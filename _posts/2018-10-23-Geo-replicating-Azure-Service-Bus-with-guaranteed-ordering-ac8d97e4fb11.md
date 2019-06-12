---
title: Geo-replicating Azure Service Bus with guaranteed ordering
description: 'Recently, I had an interesting question come across my desk from a customer:'
date: '2018-10-23T14:20:35.971Z'
categories: []
keywords: []
slug: /geo-replicating-azure-service-bus-with-guaranteed-ordering-ac8d97e4fb11
---

![Photo by [Slava Bowman](https://unsplash.com/@slavab?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](img/0__eAcdrF5a1HTPad7v.)
Photo by [Slava Bowman](https://unsplash.com/@slavab?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

Recently, I had an interesting question come across my desk from a customer:

> How do we ensure Service Bus message ordering **and** get active/active geographic availability?

We’ve solved parts of this in numerous ways over the years — Service Bus supports [paired namespaces](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-async-messaging#paired-namespaces) — but with a list of caveats that include out-of-order messages. It’s also a question of _send_ availability vs. _receive_ availability. We want our producers to always be able to send messages — that means our receivers/consumers will need to be smarter, but we’ll get into that in a bit.

In our specific scenario, global message ordering is of lesser importance than transactional or scoped message order — consider 10 transactions, 0, 1, 4, and 5 are related, 2, 3, and 6 are related, and 7, 8 and 9 are related. Those three batches can be processed in any order, but the messages _within_ the specific batch or scope need to be processed in order. We see this issue frequently when customers are integrating with legacy systems or mainframes.

For a more concrete example, let’s say I’m tracking a lot of shipments. I have many shipments and each shipment has a collection of statuses. Those statuses are serial — a package destined for Charlotte from Seattle probably wouldn’t end up in Des Moines before departing Seattle.

Here’s some data:

{% gist b740dbb6baca0c21fc52c5b85a959d56 %}

However, if I have multiple packages, I don’t really care if package A or package B has its statuses updated in my backend system first, as long as the order of the status messages for each specific package stays consistent.

Another example — change-data-capture out of a database system. If I run an INSERT after a DELETE, even though the DELETE happened first, my data isn’t going to be correct.

Back to our shipment tracking scenario. First we need a way to ensure ordering of messages within our Service Bus queue or topic. We’ll use [Message Sessions](https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-sessions) for that. Message Sessions have some other tradeoffs, however — a session ID is effectively a partition key. It means that data with that key is stickied to a particular data store. It makes sense when you think about it; messages distributed across multiple data stores would require a lot more processing server-side to collect and order.

In-order processing implies some other tradeoffs too — by definition, messages are processed in order, serially, by a single consumer at a time— no processing parallelism here (well, at least not within our individual sessions — we’ll get there soon).

Messages within a session are FIFO — but not necessarily our sessions themselves. We may have many sessions to process, which we would want to process concurrently. Remember, we only care about order guarantees _within the scope_ of our session/shipment. This way we can still ensure timely message processing without impacting our ordering requirements.

In a single Azure region this is no big deal. We have our Service Bus namespace in, say, East US, and our producers and consumers live in the same region. Full-region failures in Azure are certainly quite rare, but not completely unheard of. Service degradation or service-specific outages are still a concern too, along with dependent services (e.g., when a storage outage causes cascading failures of other services within a region). For our customer, they’re deployed into two regions in the US, and App Service, SQL DB, storage, etc are all geographically replicable in one way or another (or stateless). Service Bus was the one thing we didn’t have a compelling answer for, so we decided to build them a way to achieve it.

Let’s start with the code — it’s here: [https://github.com/jpda/azure-service-bus-active-rep](https://github.com/jpda/azure-service-bus-active-rep) — we’re going to use:

* Two Service Bus namespaces, in different regions.
* A [strongly-consistent Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels#consistency-levels) collection, as a message journal
* Message Sessions in our Service Bus queues

### Send Availability

To maintain send availability, we’re going to setup two independent Service Bus namespaces and create a session-enabled queue within each namespace. You could do this with Topics as well, if your case requires that.

Each sender will mark the session with some sort of deterministic ID or domain unique key. In our shipping scenario, a tracking number would be an excellent session key. It could also be some other unique ID that represents the scope of the messages being sent — customer ID, transaction ID, etc. Your sender should send the message to **both** queues, with identical session and message IDs.

In practice, the Session ID represents the ID of the entity you would like to ‘lock’ — because the consumers will attempt to record their work items in the Cosmos DB collection using the entity ID (session or message) as the document ID, if there is a document ID collision, the consumer will receive an exception and refetch the entity to get the latest status before beginning processing. We’ll dig into this below.

You’ll also notice in the Sender that we’re setting the ScheduledEnqueueTimeUtc to now + 15 seconds on one of the queues. We’ll address this later but it’s primarily to reduce churn and prefer a ‘primary’ region over another.

[https://github.com/jpda/azure-service-bus-active-rep/tree/master/azure-service-bus-active-sender](https://github.com/jpda/azure-service-bus-active-rep/tree/master/azure-service-bus-active-sender)

### Receive Availability

Receive availability gets tricky when you have multiple copies of the same message, especially in an integration case where you don’t have full control over the final destination of the data. In our case, we’re interacting with a mainframe (among other systems) and don’t want to add undue stress by deduplicating messages via querying the mainframe.

Instead, our consumers follow a fairly simple (albeit chatty) pattern:

* Open a MessageSession
* Create new document with SessionId or SessionId\_MessageId as the Document ID
* If we succeed at creating the document, we then update the status in the document (e.g., Status: Working)
* If we get a 409 (Conflict), we pull the latest version of the document from Cosmos and check status
* If the status is Pending, we update the status to Working, attempt to write again, wait for 200
* If we succeed, then we start processing the message
* When we’re complete, we update the document again and Complete the Service Bus message.

That’s a lot so we’ll break it down a bit.

* First we start receiving a `MessageSession`. This locks/hides all other messages with that `SessionID` in the specific queue, so we can be sure this consumer will process them in the order received.
* Before we start processing the message, we record in our journal (our Cosmos DB collection) the session ID or session/message ID composite. Because we’re using this as our document ID, if there is a conflict (e.g., another consumer has already created the document), Cosmos will return a `409 Conflict` — at which point we know another consumer has started reading the session in one of our queues.
* If that journal write is successful, we then attempt to update the document — updating a field called Status to Pending or Working.
* This update follows the same rules; if the document we have updated is older than what is in the database, Cosmos returns an error code — at which point we fetch the latest version, check status and decide from there.
* If the update to status succeeds (e.g., we’ve updated the status to Working) we can start to work.
* These two operations being discrete (instead of as a single operation) means we keep the window for changes small.
* When we’re done processing the message, we can Complete the message to remove it from the queue and begin processing the next.
* The other consumer in the secondary region will pull the document, see the status as ‘Working’ and Abandon the message; abandoning here will cause the message to unlock, and the next consumer will attempt to pick it up. When a consumer reads the message, checks status and sees a status of Completed, the message in the secondary queue will be Completed(note, in the code sample today, the consumers are not abandoning the messages, they are looping with a sleep/wait to recheck message status).

In a failure case, where one of the consumers has failed, we have two stages of failure:

The processing logic in the consumer has died, but not the process (e.g., the process hosting the processing logic is still alive) — in this case, our processor could attempt to reprocess, or in the case of an irrevocable failure, update the journal status for that message or session to Failed or Faulted, and move the message to the dead letter queue for manual remediation.

If the consumer has died completely (as in, the process is completely dead), we can use `MessageWaitTimeout` on the `SessionHandlerOptions` we use to configure the `SessionHandler` to set a reasonable timeout. Once that timeout duration has passed, the session is unlocked and our next consumer will pick up the session, check status in Cosmos and continue processing.

[https://github.com/jpda/azure-service-bus-active-rep/blob/master/azure-service-bus-active-receiver-lib/DataReceiver.cs](https://github.com/jpda/azure-service-bus-active-rep/blob/master/azure-service-bus-active-receiver-lib/DataReceiver.cs)

### Entity Locks

Your choice of entity lock has some implications. If you choose to lock at the Session level, your secondary receivers will abandon the _session_, at which point your primary consumer is expected to process the entire session. The risk here is you may need to manage which messages have been processed in the session in case of a failure — e.g., Session ID 1 has messages A B C and D. A and B process correctly, but C causes the consumer to die; the secondary consumer will need to either reprocess _all_ messages in its copy of the session (as it doesn’t know which messages have been processed), supporting at-least-once delivery, or it needs a message processing log to ensure it doesn’t process a message a second time.

If you go the session + message composite ID route, where you’re logging each message in a session, you’ll be able to keep your two queues in sync both at the session level and the message level. As messages change state within the primary queue, as the secondary processors pick up that change, they’ll dispose of the messages to mirror that (e.g., Session 1 Message A, primary has completed, journal updated, when Session 1 Message A secondary checks Cosmos with that ID and sees it is completed, it will complete the secondary message). The risk here is a potential out-of-order case where the secondary gets ahead of the primary, because of an unknown failure with the journal. I haven’t figured out a case where this would happen, as the messages should be in the same order in both queues, but theoretically:

* **Primary** Message 1 → Cosmos record written → Processing beginning
* **Secondary** Message 2 → No Cosmos record written → Processing beginning

### Load balancing

In our scenario, we have a set of legacy systems, some are single instance, some are on-premises. We’d prefer the majority of our traffic go through a single ‘primary’ Azure region, closest to our on-premises systems, but failover to a different region in case of a region fault.

In addition, this model is similar to products we’re already using (like SQL DB geo-failover), so we’d already be following this primary/secondary type of pattern in any case.

That said, this same pattern could be used for processing messages in any region, by any consumer, for potentially greater scale and throughput.

### Additional notes + thoughts

As we said earlier, the _senders_ here are dictating which entity to lock — if, for example, we want to allow individual messages to be locked (vs an entire session), the sender can use a completely unique or more granular session ID (perhaps a composite, like `PackageId_ShipmentStatusId`), which would be reflected in the journal. In this case, our primary and secondary consumers could consume the session independently in each queue. The receivers don’t care what entity is being locked, as long as the entity locking (and by extension, the entity ID generation scheme) is consistent.

We’re writing messages with `ScheduledEnqueueTimeUtc` to the secondary queue with a 15 second delay, primarily to prefer our primary region over the secondary. Provided everything is operating normally, this should provide ample time for the primary set of consumers to receive, check and record work items before the message even appears in the secondary queue.

This is a work in-progress, so feedback is welcome.
