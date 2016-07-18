---
layout:     post
title:      "Tell me a story"
subtitle:   "Storyboard's flexible logging architecture"
date:       2016-07-18 12:00:00
author:     "Guillermo Grau"
header-img: "img/02.jpg"
comments:   true
---

_**TL;DR**&nbsp;&nbsp;&nbsp;TBW [Storyboard](https://github.com/guigrpa/storyboard)._

Logging might not look like the most sexy subject to blog about. When you are building the most exciting new application, why would you even *want* to think about logging and not just… go out play Pokémon?

The fact is: sooner or later, just as with testing, you will *need* to give it some thought. In a typical client-server web application, you will start having concurrent sessions. And even at the client side, where you have only one user, simultaneous actions will start making your log less and less readable. Debugging, or just grasping what's going on, can quickly turn into a nightmare if you don't take the necessary (logging) precautions.

Even if you are not developing a webapp, you may be overwhelmed by your logs. In my current project, I'm developing a Node-only application with multiple bidirectional interfaces (an interactive SSH session, a proprietary TCP link, an interactive CLI conversation with the user) all generating events like crazy.

In an ideal world, I'd like to have an easy-to-use library that would let me filter logs by relevance or source module, attach objects, group them hierarchically in stories ("John clicks on Buy" ⊂ "John visits the store"), persist them to a file/database, send them a separate process for trend analysis… The icing on the cake? I'd love to see what's happening at the client and the server *in a single place* (how about a Chrome DevTools extension?).

Well... **have you checked out [Storyboard](https://github.com/guigrpa/storyboard) yet?**

<a href="{{ site.baseurl }}/img/Storyboard.gif">
    <img src="{{ site.baseurl }}/img/Storyboard.gif" alt="Storyboard DevTools in action">
</a>
<span class="caption text-muted">The Storyboard DevTools in action</span>

## A flexible logging architecture

Storyboard is based on a hub that receives messages and broadcasts them to all attached *listeners*, or *plugins*. Even though they are called listeners, they can also send messages to the hub, making them quite powerful.

Let's say we have a typical client-server application. You could configure Storyboard like this for development:

<a href="{{ site.baseurl }}/img/sb-typical-arch.png">
    <img src="{{ site.baseurl }}/img/sb-typical-arch.png" alt="Storyboard logging config for a client-server application">
</a>
<span class="caption text-muted">Storyboard logging configuration for a client-server application</span>

You have a server-side hub receiving server stories and relaying them to console, disk and database. In addition, server logs are sent via WebSockets to the client-side hub. At the client side, both server and client logs are sent to the console as well as to the Storyboard DevTools.

Signalling messages are exchanged between the DevTools extension and the WebSocket Server in both directions. Such signalling is used for user authentication or, more interestingly, to configure server-side log filters from the extension itself. If you want to get more logs from a particular server module, you can just tweak the filter and logs will begin pouring in, no reset required.

## Other example architectures

<a href="{{ site.baseurl }}/img/sb-node-only.png">
    <img src="{{ site.baseurl }}/img/sb-node-only.png" alt="Logging config for a Storyboard-less Node app">
</a>
<span class="caption text-muted">Logging configuration for a Storyboard-less Node application</span>

<a href="{{ site.baseurl }}/img/sb-mobile.png">
    <img src="{{ site.baseurl }}/img/sb-mobile.png" alt="Logging configuration including upload of mobile client logs">
</a>
<span class="caption text-muted">Logging configuration including upload of mobile client logs</span>


## Conclusion

After all this, which is the most suitable option?

* Don't forget the obvious: do you really need immutability? Probably yes, but use your best judgement. If performance is crucial and you can sacrifice code elegance and the other nice perks immutability brings along, well... just **mutate in peace!** I mean, *in place*.

* If you need a complete, battle-tested library and don't mind the lock-in associated to a non-native API even for reads: use **Immutable.js**.

* If you prefer plain arrays/objects, use **Timm**.

* If your typical use cases involve more reading than writing, use **Timm** as well.

* If you do a lot of writes on very long arrays or fat objects, use **Immutable.js**.
