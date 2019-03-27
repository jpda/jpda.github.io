---
title: "SharePoint Modal Dialog — Script Only"
date: 2013-02-28T06:25:00.000Z
author: "John Patrick Dandison"

---

### SharePoint Modal Dialog — Script Only

This is a personal favorite. Dead simple to implement but really allows for SharePoint customization while keeping it familiar. You can see it on the home page of this very blog.

It’s pretty simple and you can implement it without any server-side code. All you need is some javascript. The code below is in jQuery, but that’s only because jQuery makes it easy. You could do all of this without jQuery if you wanted.

#### Code

It starts with the [SP.UI.ModalDialog object](http://msdn.microsoft.com/en-us/library/ff410058%28v=office.15%29.aspx). It’s super simple to implement a vanilla modal. Here’s the example from MSDN:
`var options = new {  
                      title: &#34;Modal Title&#34;,  
                      width: 300,  
                      height: 150,  
                      url: &#34;/Lists/Posts/NewPost.aspx&#34;  
                     };   
   SP.UI.ModalDialog.showModalDialog(options);`

Which is great, but now we need to wire it up to _do_ something. While we’re at it, let’s add some flexibility.

#### Flexibility

At this point, we should make this a little more reusable. Perhaps something you can stick in a master page, if you’d like. Add some parameters &amp; now you’ve got an easy, one-line function you can call from wherever.
`function showSharePointModal(title, url){  
    var options = new {  
        title: title,  
        width: 300, //leave out and it&#39;ll auto-size  
        height: 300, //leave out and it&#39;ll auto-size  
        url: url  
    };  
    SP.UI.ModalDialog.showModalDialog(options);  
  }``showSharePointModal(&#34;Dialog Title&#34;, &#34;google.com&#34;);  

So now we&#39;ve got an easy way to show a modal, but even this is too much work. We need to find the deepest depths of our inner laziness.``A Selector``First, figure out a good jQuery selector to mark the anchor links with that you want to have open in the dialog. I&#39;ll use &#39;open-me-in-modal&#39; for this example.``IsDlg=1``This query string is your friend, whether you know it or not. It tells SharePoint that, when opening this link, it needs to strip off the primary chrome, because the page is getting stuffed into a dialog.  
_Neat._``Going A Bit Wild``Which leads us to this. We&#39;re heading off the reservation here, but it&#39;ll be fun.``//wire up the event handler to bind to your selector  
  $(document).ready(function() {   
    $(&#34;a.open-me-in-modal&#34;).on(&#34;click&#34;, function(e){  
      //let&#39;s grab the anchor&#39;s destination. we&#39;ll need it where we&#39;re going.  
      var destination = $(this).attr(&#34;href&#34;);   
      //next, we need to append the IsDlg query string. This code should be more robust, to check for  
      //things like &#39;is a relative URL?&#39; (i.e., is a SharePoint page), but you get the point.  
      if(destination.indexOf(&#34;?&#34;) &gt; -1) {  
        destination += &#34;&amp;IsDlg=1&#34;;  
      }  
      else {  
        destination += &#34;?IsDlg=1&#34;;  
      }  
      //the jQuery .text() function gets what is in between the opening and closing tags...I think.  
      var title = $(this).text();   
      var dialog = SP.UI.ModalDialog.showModalDialog({  
            title: title,  
            url: destination  
          });  
      e.preventDefault(); //important, this prevents the anchor tag from actually doing what it&#39;s supposed to do  
      //read more about why this is so important. srsly, it&#39;s like five lines down.  
    });  
  });``And to use it, you just need an anchor like this:``[Click me](/SiteAssets/BadIdeas.jpg)``At this point, any and all anchor tags with class=&#34;open-me-in-modal&#34; will open in your SharePoint dialog, keeping users on your pages longer.``Check out the MSDN page for how to do callbacks (like OK\Cancel). It&#39;s just another option to implement:``dialogReturnValueCallback: function(dialogResult, returnValue) {   
  thingsAndStuff();  
 }`

#### Why This Rocks — Graceful Degradation
`This is good because it looks good &amp; research shows users interact with dialogs more than links to other pages. But besides all of that, it&#39;s got mad graceful degradation - because it&#39;s just an anchor. Putting``e.preventDefault();``at the end of the method means that if anything blows up _before_ the``preventDefault()``then the link will just act normally &amp; take the user to that page. Great for the Googlebot, too.`
