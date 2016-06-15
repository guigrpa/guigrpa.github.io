---
layout:     post
title:      "Painless immutability"
subtitle:   "A trade-off comparison of ImmutableJS, seamless-immutable and Timm"
date:       2016-06-12 12:00:00
author:     "Guillermo Grau"
header-img: "img/02.jpg"
---

*TL;DR With the popularity of ImmutableJS and seamless-immutable going through the roof, who needs yet another immutability library? Maybe you, if you **don't want to lock your application data with a non-native API, need reasonable performance and an easy-to-use API**. That's exactly the balance Timm tries to strike.*

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

While not providing all the bells and whistles, Timm is 10 times smaller and uses much more basic tools, although it also focuses on performance. Reads are as fast as native (you don't even use the library at all), and writes are also reasonably fast (see [these benchmarks](https://github.com/guigrpa/timm#benchmarks) and the figure below):

* On small arrays/objects: same performance as ImmutableJS for shallow writes, and becoming increasingly faster than ImmutableJS for deeper writes.

* On large arrays or objects (see *Array: write* in the figure below, where arrays have 1000 elements): increasingly slower than ImmutableJS as arrays become larger or objects have more properties. This is where ImmutableJS really shines!

* Faster (even *extremely* faster) than [seamless-immutable](https://github.com/rtfeldman/seamless-immutable), which also uses native objects.

<a href="https://github.com/guigrpa/timm#benchmarks">
    <img src="{{ site.baseurl }}/img/timm-benchmarks.png" alt="Performance comparison">
</a>
<span class="caption text-muted">Read/write benchmarks for ImmutableJS, Timm and seamless-immutable</span>


## ...and what about the ES6/ES7 spread operator?

Finally, why not use ES6's shiny new array spread operator and the its upcoming ES7 sibling for objects? After all, native syntax can't be beaten, right? Why not do the following?

```js
const obj = { a: 3, b: 5 };

// Why not do this...
const obj2 = { ...obj, b: 6 };

// ...instead of this?
const obj2 = timm.merge(obj, { b: 6 });
```

The first one is clearly more convenient. However, beware:

```js
const obj = { a: 3, b: 5 };

// ES7 when nothing changes:
{ ...obj, b: 5 } === obj
// -> false

// Timm when nothing changes:
timm.merge(obj, { b: 5 }) === obj
// -> true!
```

Wait, what's up? 
The reason is quite subtle. If you use the former, you will *always* be generating new objects even if the operation doesn't mutate the object at all (e.g. `obj2 = { ...obj, b: 5 };`. By contrast, Timm will lazily instantiate a new object when it detects that the object is going to be modified; hence, you'll create fewer objects unnecessarily.


## Conclusion

With the popularity of ImmutableJS and seamless-immutable going through the roof, who needs yet another immutability library? Maybe you, if you **don't want to lock your application data with a non-native API, need reasonable performance and an easy-to-use API**. That's exactly the balance Timm tries to strike.
