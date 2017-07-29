---
layout: post
title: "Notes Panel for Adobe Audition Development Log"
description: "My development log for my first Adobe Audition Extension"
date: 2017-07-29
tags: [CEP, CEF, Extensions, Development, Audition]
comments: true
share: true
---
After quite a bit of work, and fine tuning, I have created the Notes extension for Adobe Audition, on the surface it may not sound particularly complicated but, there are a lot of cheats and hacks going on behind the scenes to make this extension work they way it does, in this blog post I hope to go through some of the issues I found while developing the extension.

First off I would like to point out a few things:

* Audition only recently received support for CEP extensions with version CC 2015.1, thus, the extendscript API in Audition is also really new, so new in fact that new extendscript functions are being added with every release. As well there is hardly any documentation on CEP extensions or the extendscript API for Audition.
*  I would like to extend a huge thank you to Adobe Audition developer [Charles Van Winkle](https://twitter.com/AudiblyChuck) who helped point me in the right direction while developing the extension!
*  This post is not meant to be a tutorial for creating an extension for Adobe Audition, it's more of a "What I learned" type post. I might make a tutorial post on extensions in the future though!

When I got the idea for my extension I came up with a few different ways of storing notes, I wanted it to be out of sight for the end user, so saving a .txt file in the same directory was out of the question(would have been a bit to easy for my taste as well). My first idea was to use extendscript to find the current document's location on the user's system and then modify the .sesx file (which is just an XML file btw, like a lot of Adobe file formats) with the notes, this, however, would limit the notes to just Audition Sessions and would risk project corruptions if something went wrong. My last idea was XMP metadata, seeing as how Audition already had strong support for XMP it seemed like a great idea. But the only way to access the XMP metadata was using the extendscript DOM, which has little to no communication with the Javascript DOM that the CEP extension system uses. I found my solution by way of the CEP Samples repository on GitHub, more specifically the XMPSamplePanel extension, which has some useful extendscript files. The one I ended up "stealing" was _XMPBridge.jsx_ which worked as a proxy layer between the extendscript XMP API and the CEP javascript DOM, using that I was able to read and modify XMP metadata. The end result was a bit of a bodge, but it worked! 

My next problem was saving the notes, so far I had solved the problem with a button inside the panel, but I felt forcing the user to move their mouse over to save their notes every other second was a bit too much. So I thought to have the extension save whenever the active document was saved was a good idea. After a quick look in the CEP Cookbook, I could see that the _documentAfterSave_ CEP event would enable me to do just that, but for reasons I don't know (probably the cookbook being wrong, as is usually the case...) it didn't work, after trying to get the event to intercept i ended up giving up and went for a different more "hacky" approach. I decided to create my own custom CEP event, that would simply intercept the extendscript Audition save event and send it to my extension. To my delight, this actually worked, a bit more "hacky" than I would have liked, but good enough for the time being. 

Next up I wanted the extension to refresh the notes when the active document was changed, once again I turned to CEP events, and found that the _documentAfterActivate_ event actually worked like it was supposed to so that one was pretty easy to setup. 

All that, just to create a simple notes panel for Adobe Audition. I won't lie I was tempted to rip my own hair out at times, but the feeling of success whenever I actually got the result I wanted was incomparable to anything else. I guess that's why I keep coming back to extension development, it might be difficult at times, but other times it is really rewarding. 

The extension is available on the Adobe Exchange marketplace for the nifty price of 0$: [https://exchange.adobe.com/addons/products/19469](https://exchange.adobe.com/addons/products/19469). If you are interested I posted the source code for the extension to GitHub here: [https://github.com/Hennamann/Notes-Panel-for-Adobe-Audition](https://github.com/Hennamann/Notes-Panel-for-Adobe-Audition).

Thank you for reading this post, feel free to ask your questions in the comments below, and if you want more updates from me I would recommend following me on [Twitter](https://twitter.com/henrikstabell) and to subscribe to my blog here: [https://blog.henrikstabell.com/feed.xml](https://blog.henrikstabell.com/feed.xml).