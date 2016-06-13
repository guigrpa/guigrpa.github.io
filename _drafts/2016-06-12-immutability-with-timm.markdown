---
layout:     post
title:      "Painless immutability"
subtitle:   "Timm: a tiny immutability library with reasonable trade-offs"
date:       2016-06-12 12:00:00
author:     "Guillermo Grau"
header-img: "img/02.jpg"
---

We've read the posts, watched the videos, understood the concepts. Immutability can be really helpful in some scenarios:

* We perform some lengthy computations on data and want to memoize the results, reusing them until inputs change.

* We want to implement an Undo/Redo functionality in a really simple way.

* We want to untangle your application's state and use something like Redux with time travel, serialization, rehydration, you name it.

* We are stuck with our React application performance and have heard of `shouldComponentUpdate`'s wonders.

In all these cases, using immutable data is the way to go. It can make code more predictable, allow trivial object comparisons <<<<TBW>>>>>...

But using immutable operations does not come without its own set of drawbacks. It will never be as fast as in-place mutation, no matter how hard you try. You will need to bend your mind to implement some algorithms, especially recursive ones. And it can be a pain to work with in some cases, depending on the immutability tools you work with.

But what if we found a way to alleviate the cognitive burden on the developer, while allowing her to reap the benefits of immutability?

## Trade-off #1: Friction-less immutability


 does not make it the panacea, however; use your best judgement 



######
Links to places singing the virtues of immutability. Redux, shouldCompUpdate, etc etc.

present Timm, inspired by eggheadio season 1

Some advantages wrt ES6: more complete, transparent when no change

Comparison with ImmJS: Faster in many cases, less friction. Tiny. Less lock-in, plain data.

Limitations: can't protect data. Not opaque

Why not simply use ES6 rest operators?
