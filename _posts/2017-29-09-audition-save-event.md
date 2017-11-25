---
layout: post
title: "Properly intercepting the Audition document save event"
description: "A follow-up to my Audition notes panel dev log"
date: 2017-09-29
tags: [CEP, CEF, Extensions, Development, Audition]
comments: true
share: true
---

It's been a while since my last post, not because nothing has happened since but because I simply did not have the time to write new posts. So I come today with great news!

In my last post exactly three months ago, I talked about the development of my Notes Panel for Adobe Audition extensions and all the issues I ran into. The issue I was stuck on for the longest time was intercepting the event for when a user saved the current document. I never figured out the proper way and instead came up with a more hacky way of doing it using extendscript. But since then I have with the help of some awesome Audition developers figured out how to get the event to work! And I aim to share it with you in this post.

But first, let's explain what should work and how I got it to work in the extension initially.

CEP extensions along with extendscript have events that you can intercept, extendscript has a lot, and most of them are unique to each app. CEP has a few of its own that is shared between applications, like opening a document, saving a document, changing the active document, etc. Some of the Adobe apps support all of them while others just support a few of them, heck some don't support them at all. This is all documented in the CEP cookbook. But as I mentioned in my last post the event for saving a document never fires, and if it does, it is impossible to intercept it.

So how did I circumvent it in my extension? By making my own event of course. Adobe provides an extendscript library called PlugPlugExternalObject which among other things allows you to create custom CEP events that can then be intercepted using the CSInterface like a normal CEP event. Here's an example similar to what I did in the Notes Panel extension:

In Extendscript:

```javascript
function addDocumentListener() {
    app.activeDocument.addEventListener(DocumentEvent.EVENT_SAVED, documentSaved); //Adds a extendscript event listener for auditions save event.
    
    function documentSaved(event) { // Creates a custom CEP event that is fired once the extendscript event is intercepted.
    var csxsEvent = new CSXSEvent();
    csxsEvent.type = "DocumentSaved";
    csxsEvent.data = "Audition Document Saved!";
    csxsEvent.dispatch();
}
```

In the extension Javascript file:

```javascript
csInterface.evalScript("addDocumentListener()"); // Run the extendscript code for adding the event listener.     
csInterface.addEventListener("DocumentSaved", function (event) {  // Intercept our custom CEP Event.
   console.log("Document Saved!");
})
```

This works, but it has a few nasty side effects:

1. Since the notes are saved as metadata Audition detects changes to metadata immediately after saving the document, leading to the dreaded "Unsaved changes" Asterisk(*) being present at all times when using the panel.
2. Going trough extendscript and then through the CEP APIs adds more points of failure and while I have not tested this could also take longer than a more native CEP event would.

So what is the proper way to do it then? After talking to a few Audition developers, I finally know the answer:

Javascript code:
 
```javascript
var csInterface = new CSInterface();

csInterface.addEventListener("com.adobe.csxs.events.DocumentSaveAs", function (event) {
console.log("com.adobe.csxs.events.DocumentSaveAs");
});
```

Yup that is it, but with no actual documentation even mentioning this event existing, one can hardly blame me or anyone else for not figuring it out on their own, unless I'd try every conceivable name for a save event.

So that's it, after three months a rather simple issue has been resolved. I hope it helped you as much as it would have helped me back when I was frantically searching the internet for this straightforward answer! 

Thank you for reading this post, feel free to ask your questions in the comments below, and if you want more updates from me, I would recommend following me on [Twitter](https://twitter.com/henrikstabell) and to subscribe to my blog here: [https://blog.henrikstabell.com/feed.xml](https://blog.henrikstabell.com/feed.xml).