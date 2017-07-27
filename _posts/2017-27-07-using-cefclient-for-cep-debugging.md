---
layout: post
title: "Using cefclient for CEP Extension Debugging"
description: "Among the million things i hate about CEP extension development, debugging has been one of my biggest complaints with it. Until today. "
date: 2017-07-27
tags: [CEP, CEF, Extensions, Development]
comments: true
share: true
---

Among the million things i hate about CEP extension development, debugging has been one of my biggest complaints with it. Until today. 

The issues i have with the current "intended" way of debugging extensions is the fact that it requires me to run the debugging tools as a tab in Chrome, i understand why it´s difficult to implement it in the actual host app, and the intended way makes sense and it works for the most part. But i am not a fan of being restricted to a web browser when debugging, i use Chrome for a lot more than just debugging CEP extensions so i often find myself getting lost in a million different tabs, thereby wasting precious development time. And i can only imagine how annoying it must be to use another browser than Chrome, and then have to install Chrome purely to debug CEP extensions.

So i have spent some time now looking for a solution, this searching lead me to the Chrome DevTools App project: [https://github.com/auchenberg/chrome-devtools-app](https://github.com/auchenberg/chrome-devtools-app) simply put, it's the DevTools from chrome put into a standalone electron app. I was really hopeful seeing as how it used the remote debugging functionality of Chrome and CEF(Chromium Embedded Framework), which from the little information given to us by Adobe is the same way that CEP debugging works! But despite trying for a few hours i was unable to get my extensions to show up in the DevTools App, not sure if this is on Adobe's side or on the DevTools side, though at this point i honestly didn't care as fixing it would probably require more work than it was worth. It should also be noted that Chrome DevTools App hasn't been updated for about 2 years, so it's missed out on a lot of updates to the Chrome Dev Tools. 

And so it was back to Google, which lead me to this blog post from fellow extension developer:  RorohikoKris: [http://coppieters.nz/?p=151](http://coppieters.nz/?p=151) where he mentioned "hacking" the cefclient found here: [http://opensource.spotify.com/cefbuilds/index.html](http://opensource.spotify.com/cefbuilds/index.html) to use it for CEP extension debugging. I tried the hack and it worked wonders, but i also took note of the bottom of his post where he mentioned creating command line arguments for opening a specific url instead of the default url, so along with downloading the actual cefclient app i also downloaded the source code to have a look at creating that command line argument. After some quick rummaging in the C++ code i found this: 

```cpp
// Copyright (c) 2013 The Chromium Embedded Framework Authors. All rights
// reserved. Use of this source code is governed by a BSD-style license that
// can be found in the LICENSE file.

#include "tests/shared/common/client_switches.h"

namespace client {
namespace switches {

// CEF and Chromium support a wide range of command-line switches. This file
// only contains command-line switches specific to the cefclient application.
// View CEF/Chromium documentation or search for *_switches.cc files in the
// Chromium source code to identify other existing command-line switches.
// Below is a partial listing of relevant *_switches.cc files:
//   base/base_switches.cc
//   cef/libcef/common/cef_switches.cc
//   chrome/common/chrome_switches.cc (not all apply)
//   content/public/common/content_switches.cc

const char kMultiThreadedMessageLoop[] = "multi-threaded-message-loop";
const char kExternalMessagePump[] = "external-message-pump";
const char kCachePath[] = "cache-path";
const char kUrl[] = "url";
const char kOffScreenRenderingEnabled[] = "off-screen-rendering-enabled";
const char kOffScreenFrameRate[] = "off-screen-frame-rate";
const char kTransparentPaintingEnabled[] = "transparent-painting-enabled";
const char kShowUpdateRect[] = "show-update-rect";
const char kMouseCursorChangeDisabled[] = "mouse-cursor-change-disabled";
const char kRequestContextPerBrowser[] = "request-context-per-browser";
const char kRequestContextSharedCache[] = "request-context-shared-cache";
const char kBackgroundColor[] = "background-color";
const char kEnableGPU[] = "enable-gpu";
const char kFilterURL[] = "filter-url";
const char kUseViews[] = "use-views";
const char kHideFrame[] = "hide-frame";
const char kHideControls[] = "hide-controls";
const char kHideTopMenu[] = "hide-top-menu";
const char kWidevineCdmPath[] = "widevine-cdm-path";
const char kSslClientCertificate[] = "ssl-client-certificate";
const char kCRLSetsPath[] = "crl-sets-path";

}  // namespace switches
}  // namespace client
```

It's a list of command line arguments available in cefclient, and wouldn't you know it, one of them does exactly what i need:

```cpp
const char kUrl[] = "url";
```
And so running the cefclient executable from the command line using something like this:
`/Users/yourusername/Applications/cefclient.app/Contents/MacOS/cefclient --url=http://localhost:8089` 
Will open the cefclient app and immediately load the url: `localhost:8089` which in this case is the debug url i have set for my extension. This also circumvents the 8 character limitation found in RorohikoKris's hack, meaning you can use a port number thats longer than 4 numbers! 

From here it would be pretty easy to use nodejs in our extension and open the cefclient app with the current debug url using the child process module, by pressing a button in our extension. I haven't gotten around to doing this yet, but i may make a new post detailing how to do this in the future, feel free to "pester" me in the comments if you want me to do that!

At this point i only had one minor nuisance left, which was the icon for the cefclient app, looking rather dull. So i decided to spend some time creating a new icon that would look good in my dock. ![](/images/cepdebugdock.png) I decided to base the icon off of the official Chrome Dev Tools logo:

![](http://mikeking.io/devtools-author/images/chrome_devtools_256px.png)

For those interested here's a download link for the Adobe illustrator file: [https://assets.adobe.com/link/b24d9b35-bd92-4717-62da-595c06f5b717](https://assets.adobe.com/link/b24d9b35-bd92-4717-62da-595c06f5b717) and here's a link for the mac .ICNS file: [https://assets.adobe.com/link/9dd34622-3d77-4717-725e-c7aa1e47203b](https://assets.adobe.com/link/9dd34622-3d77-4717-725e-c7aa1e47203b.)

Hope you found the information i shared in this blog post useful, if you did i would really appreciate it if you could share it with your friends on Facebook and Twitter! And if you want to stay tuned to future posts on my blog, subscribe to my blog feed here: [https://blog.henrikstabell.com/feed.xml](https://blog.henrikstabell.com/feed.xml).

