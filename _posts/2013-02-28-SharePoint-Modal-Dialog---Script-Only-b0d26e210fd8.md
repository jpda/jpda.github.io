---
title: SharePoint Modal Dialog — Script Only
description: ''
date: '2013-02-28T06:25:00.000Z'
categories: []
keywords: []
slug: /sharepoint-modal-dialog-script-only-b0d26e210fd8
---

This is a personal favorite. Dead simple to implement but really allows for SharePoint customization while keeping it familiar. You can see it on the home page of this very blog.

It’s pretty simple and you can implement it without any server-side code. All you need is some javascript. The code below is in jQuery, but that’s only because jQuery makes it easy. You could do all of this without jQuery if you wanted.

#### Code

It starts with the [SP.UI.ModalDialog object](http://msdn.microsoft.com/en-us/library/ff410058%28v=office.15%29.aspx). It’s super simple to implement a vanilla modal. Here’s the example from MSDN:

var options = new {  
                      title: "Modal Title",  
                      width: 300,  
                      height: 150,  
                      url: "/Lists/Posts/NewPost.aspx"  
                     };   
   SP.UI.ModalDialog.showModalDialog(options);

Which is great, but now we need to wire it up to _do_ something. While we’re at it, let’s add some flexibility.

#### Flexibility

At this point, we should make this a little more reusable. Perhaps something you can stick in a master page, if you’d like. Add some parameters & now you’ve got an easy, one-line function you can call from wherever.

function showSharePointModal(title, url){  
    var options = new {  
        title: title,  
        width: 300, //leave out and it'll auto-size  
        height: 300, //leave out and it'll auto-size  
        url: url  
    };  
    SP.UI.ModalDialog.showModalDialog(options);  
  }

showSharePointModal("Dialog Title", "google.com");  
  
So now we've got an easy way to show a modal, but even this is too much work. We need to find the deepest depths of our inner laziness.

A Selector

First, figure out a good jQuery selector to mark the anchor links with that you want to have open in the dialog. I'll use 'open-me-in-modal' for this example.

IsDlg=1

This query string is your friend, whether you know it or not. It tells SharePoint that, when opening this link, it needs to strip off the primary chrome, because the page is getting stuffed into a dialog.  
_Neat._

Going A Bit Wild

Which leads us to this. We're heading off the reservation here, but it'll be fun.

//wire up the event handler to bind to your selector  
  $(document).ready(function() {   
    $("a.open-me-in-modal").on("click", function(e){  
      //let's grab the anchor's destination. we'll need it where we're going.  
      var destination = $(this).attr("href");   
      //next, we need to append the IsDlg query string. This code should be more robust, to check for  
      //things like 'is a relative URL?' (i.e., is a SharePoint page), but you get the point.  
      if(destination.indexOf("?") > -1) {  
        destination += "&IsDlg=1";  
      }  
      else {  
        destination += "?IsDlg=1";  
      }  
      //the jQuery .text() function gets what is in between the opening and closing tags...I think.  
      var title = $(this).text();   
      var dialog = SP.UI.ModalDialog.showModalDialog({  
            title: title,  
            url: destination  
          });  
      e.preventDefault(); //important, this prevents the anchor tag from actually doing what it's supposed to do  
      //read more about why this is so important. srsly, it's like five lines down.  
    });  
  });

And to use it, you just need an anchor like this:

[Click me](/SiteAssets/BadIdeas.jpg)

At this point, any and all anchor tags with class="open-me-in-modal" will open in your SharePoint dialog, keeping users on your pages longer.

Check out the MSDN page for how to do callbacks (like OK\\Cancel). It's just another option to implement:

dialogReturnValueCallback: function(dialogResult, returnValue) {   
  thingsAndStuff();  
 }

#### Why This Rocks — Graceful Degradation

This is good because it looks good & research shows users interact with dialogs more than links to other pages. But besides all of that, it's got mad graceful degradation - because it's just an anchor. Putting

e.preventDefault();

at the end of the method means that if anything blows up _before_ the

preventDefault()

then the link will just act normally & take the user to that page. Great for the Googlebot, too.