---
layout:     post
title:      "Untangling spaghetti logs"
subtitle:   "Storyboard's flexible logging architecture"
date:       2016-07-18 12:00:00
author:     "Guillermo Grau"
header-img: "img/04-books.jpg"
comments:   true
---

**_TL;DR_ &nbsp;&nbsp;&nbsp;Logging might not be super-exciting, but when you hit your head against a wall during debugging, you sometimes wish you'd used better tools. [Storyboard 2.0](https://github.com/guigrpa/storyboard) has just been released and provides a flexible architecture, a powerful feature set and a novel approach to ~~storytelling~~ logging.**

OK, so it might not look like the most sexy subject to blog about. When you are building the most exciting new application, why would you even *want* to think about logging and not just… go out play Pokémon?

The fact is: sooner or later, just as with testing, you will *need* to give it some thought. In a typical client-server web application, you will start having concurrent sessions. And even at the client side, where you have only one user, simultaneous actions will start making your logs less and less readable. Debugging, or just grasping what's going on, can quickly turn into a nightmare if you don't take the necessary precautions.

Even if you are not developing a web app, you may be overwhelmed by your logs. In my current project, I'm developing a Node-only application with multiple bidirectional interfaces (an interactive SSH session, a proprietary TCP link, an interactive CLI conversation with the user), all of them generating events like crazy.

In an ideal world, I'd like to have an easy-to-use library that would let me filter logs by relevance or source module, attach objects, group them hierarchically in stories ("John clicks on Buy" + "Goes to check-out" ⊂ "John visits the store"), persist them to a file/database, send them to a separate process for trend analysis… The icing on the cake? I'd love to see what's happening at the client and the server *in a single place* (how about a Chrome DevTools extension?).

Well... **have you checked out [Storyboard](https://github.com/guigrpa/storyboard) yet?**

<a href="{{ site.baseurl }}/img/Storyboard.gif">
    <img src="{{ site.baseurl }}/img/Storyboard.gif" alt="Storyboard DevTools in action">
</a>
<span class="caption text-muted">The Storyboard DevTools in action</span>


## A flexible logging architecture

Storyboard is based on a hub that receives messages and broadcasts them to all attached *listeners*, or *plugins*. Despite their name, listeners can also send messages to the hub, making them quite powerful.

Let's say we have a typical client-server application. You could configure the isomorphic Storyboard library like this for development:

<a href="{{ site.baseurl }}/img/sb-typical-arch.png">
    <img src="{{ site.baseurl }}/img/sb-typical-arch.png" alt="Storyboard logging config for a client-server application">
</a>
<span class="caption text-muted">Storyboard logging configuration for a client-server application</span>

In this architecture, the server-side hub receives stories and relays them to console, disk and database. In addition, server logs are sent via WebSockets to the client-side hub, which relays server and client stories to the console as well as to the Storyboard DevTools.

Signalling messages are exchanged between the DevTools extension and the WebSocket Server in both directions. Such messages are required for user authentication or, more interestingly, to configure server-side log filters from the extension itself. If you want to get more logs from a particular server module, you can just tweak the filter (say, from `*:INFO` to `*:INFO, suspect:DEBUG`) and logs from the `suspect` module will begin to pour in, no reset required.


## Other example architectures

In some cases, you might not be able to include Storyboard in your existing project because your application might not be written in JavaScript, or because you simply don't want to change it. In this scenario, even though some benefits of the library will be lost, you can still use the Storyboard [CLI](https://github.com/guigrpa/storyboard#cli-tool), which wraps your application's default input/output streams (`stdin`, `stdout`, `stderr`):

<a href="{{ site.baseurl }}/img/sb-with-adapter.png">
    <img src="{{ site.baseurl }}/img/sb-with-adapter.png" alt="Logging configuration for a Node-only application with the Storyboard CLI tool">
</a>
<span class="caption text-muted">Logging configuration for a Node-only application with the Storyboard CLI tool</span>

This even works with a shell! Just try `sb --server bash` and you're instantly sharing your shell session with others via HTTP.

As a final example, here is a Storyboard configuration with support for mobile clients. In this case, the WebSocket client is configured with `uploadClientStories: true`. Uploaded stories will not pollute the server's logs written to console or file, but they will get saved to the database for postprocessing, and re-broadcast to other clients with the Storyboard DevTools extension.

<a href="{{ site.baseurl }}/img/sb-mobile.png">
    <img src="{{ site.baseurl }}/img/sb-mobile.png" alt="Logging configuration including upload of mobile client logs">
</a>
<span class="caption text-muted">Logging configuration including upload of mobile client logs</span>


## Conclusion

Logging is something we should keep in mind when developing an application, be it large or small, same as testing, security and other cross-cutting concerns. The flexible architecture of Storyboard may help you untangle your logs and squeeze the most out of them!
