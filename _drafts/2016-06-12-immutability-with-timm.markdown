---
layout:     post
title:      "Painless immutability"
subtitle:   "A trade-off comparison of ImmutableJS, seamless-immutable and Timm"
date:       2016-06-12 12:00:00
author:     "Guillermo Grau"
header-img: "img/02.jpg"
---

We've read [the](http://jlongster.com/Using-Immutable-Data-Structures-in-JavaScript) [posts](http://redux.js.org/docs/introduction/ThreePrinciples.html), watched the [videos](https://youtu.be/I7IdS-PbEgI), understood the [concepts](https://en.wikipedia.org/wiki/Immutable_object). Immutability can be really useful in many scenarios; some examples:

* We perform some lengthy computations on data and want to memoize the results, reusing them until inputs change.

* We want to implement an Undo/Redo functionality in an elegant way.

* We want to untangle our application's state and use something like Redux with time travel, serialization, rehydration, you name it.

* We are stuck with our React application performance and have heard of `shouldComponentUpdate`'s wonders.

In all of these cases, using immutable data is probably the way to go. It can make code more predictable, it allows trivial object comparisons, and it feels right at home with functional programming. What else could we ask for?

Well, as it turns out using immutable operations does not come without its own set of drawbacks. It will never be as fast as in-place mutation, no matter how hard you try. You will need to bend your mind to implement some algorithms, especially recursive ones. And it can be a pain to work with in some cases, depending on the immutability tools you work with.

But what if we found a way to alleviate the cognitive burden on the developer, keep performance reasonable, and allow us to reap the benefits of immutability? Enter [Timm](https://github.com/guigrpa/timm), a tiny library that tries to solve some of these trade-offs.


## Trade-off #1: seamlessness vs. protection

[ImmutableJS](http://facebook.github.io/immutable-js) is one of the big names in the immutability arena, a rock-solid library that offers you everything you'd *ever* need. Performance is also excellent, considering the many things it does, and its objects (Maps, Lists and many others) are opaque, so they're protected against your accidental mutations.

However, after using it in several projects I've always felt the friction of mixing ImmutableJS's Maps and Lists with native JS objects and arrays. It'd be impossible to count how many times I got strange runtime errors because I forgot to use Map's getter (`picture.get('url')`), instead of plain old dot notation (`picture.url`), or a warning because I used the familiar array `length` property instead of List's `size`.

So: *seamless or protected*? Timm chooses *seamless*. It would be possible to have native *and* protected objects (via `Object.freeze()`), but there seems to be some performance bottlenecks (or alternatively [different behaviours](https://github.com/rtfeldman/seamless-immutable#performance) in Development and in Production, which seems just as bad). In conclusion: you *can* accidentally mutate your objects (watch out!), but:

* You will be able to access your data in a straightforward way; and
* You won't be locked in to a particular library, for something as ubiquitous as data management.


## Trade-off #2: simplicity/size vs. write performance

ImmutableJS aims at being both comprehensive and performant. Optimising read/write speeds in immutable operations while keeping an attractive API is no easy task, and ImmutableJS uses advanced techniques ([persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure) and [structural sharing](https://en.wikipedia.org/wiki/Hash_array_mapped_trie)) to achieve just that.

While not providing all the bells and whistles, Timm is 10 times smaller and uses much more basic tools, although it also focuses on performance. Reads are as fast as native (you don't even use the library at all), and writes are reasonably fast, depending on  (see [these benchmarks](https://github.com/guigrpa/timm#benchmarks)):

* On small arrays/objects: same performance as ImmutableJS for shallow writes, and becoming increasingly faster than ImmutableJS for deeper writes.
* On large arrays or objects (see *Array: write* in the figure below, where arrays have 1000 elements): increasingly slower than ImmutableJS as arrays become larger or objects have more properties. This is where ImmutableJS really shines!
* Faster (even *extremely* faster) than [seamless-immutable](https://github.com/rtfeldman/seamless-immutable), which also uses native objects.

<a href="https://github.com/guigrpa/timm#benchmarks">
    <img src="{{ site.baseurl }}/img/timm-benchmarks.png" alt="Performance comparison">
</a>
<span class="caption text-muted">Read/write benchmarks for ImmutableJS, Timm and seamless-immutable</span>


## Conclusions

<< Why not just ES6 with accepted array rest operator and proposed object rest attributes? >>

######
Links to places singing the virtues of immutability. Redux, shouldCompUpdate, etc etc.

Some advantages wrt ES6: more complete, transparent when no change

Why not simply use ES6 rest operators?
