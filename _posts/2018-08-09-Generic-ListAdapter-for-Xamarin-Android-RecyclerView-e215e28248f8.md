---
title: Generic ListAdapter for Xamarin.Android RecyclerView
description: >-
  I’ll preface this to say I know literally nothing about Android (or mobile, in
  general) development. I’ve used Xamarin for about 7 hours…
date: '2018-08-09T16:23:14.293Z'
categories: []
keywords: []
slug: /generic-listadapter-for-xamarin-android-recyclerview-e215e28248f8
---

I’ll preface this to say I know literally nothing about Android (or mobile, in general) development. I’ve used Xamarin for about 7 hours now and I think it’s neat, but my only goal is to get up a stub of an app to act as a client for the real work, which is a bunch of Azure stuff. This might be useful to someone or it could be a complete [Dunning-Kruger](https://en.wikipedia.org/wiki/Dunning%E2%80%93Kruger_effect) moment for me, only time will tell. Proceed with caution.

After reading a bit about ListViews I found out about [RecyclerView](https://blog.xamarin.com/recyclerview-highly-optimized-collections-for-android-apps/), which has some cool viewport management stuff to reclaim memory for items you’ve scrolled past while queuing up new items about to come into viewport.

![Image from Xamarin blog post [here](https://blog.xamarin.com/recyclerview-highly-optimized-collections-for-android-apps/)](img/1__rNlpuO__byTAYObhua4IN9w.png)
Image from Xamarin blog post [here](https://blog.xamarin.com/recyclerview-highly-optimized-collections-for-android-apps/)

Anyway, I have two collection types for my stub app, Contacts and Conversations. It’s pretty straightforward, but as I was working through the adapter → viewholder stuff, it seems duplicative.

### Let’s start

I went though the Xamarin blog post, adapting what was there to my needs and types. I’ve got:

* [Contact](https://gist.github.com/jpda/4ccc9cf61210970753925262eca42954#file-contact-cs)— my actual entity, in this case `Name` and `Email`
* [ContactsListRow](https://gist.github.com/jpda/4ccc9cf61210970753925262eca42954#file-contactlistrow-axml)— the axml item template markup
* [ContactsAdapter](https://gist.github.com/jpda/4ccc9cf61210970753925262eca42954#file-contactsadapter-cs)— the adapter that actually inflates the view and binds the data
* [ContactAdapterViewHolder](https://gist.github.com/jpda/4ccc9cf61210970753925262eca42954#file-contactadapterviewholder-cs)— a holder object that keeps references to the views within the `RecyclerView` template (e.g., the `TextView`)

As I started replicating this for a different collection, it dawned on me this is actually a lot of the same code.

If we look at the ContactsAdapter, we see a lot of the same thing:

* A generic collection of our items
* Linking a `ViewHolder` to a layout
* Binding an entity item to a `ViewHolder`

That’s really about it. Take a look:

{% gist 082552dbe75062980fd9760161e2ed99 %}

Notice there’s really not a whole lot of specific stuff in here. Let’s look at the reusable one:

{% gist ad3730c22ac666b0e50d85acf313e6c9 %}

The only specific stuff we need to know about is

* `T` — Type of collection
* `V` — Type of ViewHolder
* `Action<T,V>` — a method that takes our item and our viewholder and binds them

Let’s look at usage — you’ll notice we really just pushed some code around to different places. Here’s our usage in something like MainActivity.cs. Our binding logic has moved to the anonymous method, and we’re explicitly telling our `ListAdapter` the type of our collection and our `ViewHolder`.

{% gist 349d06a233f8382000c68f46813724c8 %}

So far I think Xamarin is pretty neat. I don’t foresee a career shift to mobile development anytime soon but this is at least making it a lot easier! More to come on the Azure bits we’re wiring up here soon.
