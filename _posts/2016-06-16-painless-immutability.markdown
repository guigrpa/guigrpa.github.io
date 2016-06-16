---
layout:     post
title:      "Painless immutability"
subtitle:   "A trade-off comparison of Immutable.js, Seamless-immutable and Timm"
date:       2016-06-16 12:00:00
author:     "Guillermo Grau"
header-img: "img/02.jpg"
comments:   true
---

_**TL;DR**&nbsp;&nbsp;&nbsp;With the popularity of Immutable.js and Seamless-immutable going through the roof, who needs yet another immutability library? Maybe you, if you **don't want to lock read access to your data through a non-native API, need reasonable write performance and an easy-to-use API**. That's exactly the balance [Timm](https://github.com/guigrpa/timm) tries to strike._

We've read [the](http://jlongster.com/Using-Immutable-Data-Structures-in-JavaScript) [posts](http://redux.js.org/docs/introduction/ThreePrinciples.html), watched the [videos](https://youtu.be/I7IdS-PbEgI), understood the [concepts](https://en.wikipedia.org/wiki/Immutable_object). Immutability can be really useful in many scenarios; for example:

* We perform some lengthy computations on data and want to memoize the results, reusing them until inputs change.

* We want to implement an Undo/Redo functionality in an elegant way.

* We want to untangle our application's state and use something like Redux with time travel, serialization, rehydration, you name it.

* We're stuck with our React application performance and have heard of `shouldComponentUpdate`'s wonders.

In all of these cases, using immutable data is probably the way to go. It can make code more predictable, allows trivial object comparisons, and feels right at home with functional programming. What else could we ask for?

As it turns out, immutable operations don't come without their own drawbacks. They'll never be as fast as in-place mutations, no matter how hard you try. You'll need to bend your mind around some algorithms, especially recursive ones. And it may become a pain to work with in some cases, depending on the tools/libraries you work with.

But what if we found a way to alleviate the cognitive burden on the developer, keep performance reasonable, and allow us to reap the benefits of immutability? Enter [Timm](https://github.com/guigrpa/timm), a tiny library that tries to solve some of these trade-offs.


## Trade-off #1: seamlessness vs. protection

[Immutable.js](http://facebook.github.io/immutable-js) is one of the big names in the immutability arena, a rock-solid library that offers you everything you'll *ever* need and then some more. Performance is also excellent, considering the many things it does, and its objects (Maps, Lists and many others) are opaque, so they're protected against your accidental mutations.

However, after using it in several projects I've always felt the friction of mixing Immutable.js' Maps and Lists with native JS objects and arrays. It'd be impossible to count how many times I got strange runtime errors because I forgot to use Map's getter (`image.get('url')`), instead of plain old dot notation (`image.url`), or a warning because I took the familiar array `length` property instead of List's `size`.

So: *seamless or protected*? Timm goes for *seamless*. It would be possible to have native *and* protected objects via `Object.freeze()`, but there seems to be some performance bottlenecks (or alternatively [different behaviours](https://github.com/rtfeldman/seamless-immutable#performance) in Development and Production mode, which seems just as bad). In conclusion: you *can* accidentally mutate your objects (watch out!), but:

* You will be able to access your data in a straightforward way; and

* You won't be locked in to a particular library, for something as ubiquitous as data management.


## Trade-off #2: simplicity/size vs. write performance

Immutable.js aims at being both comprehensive and performant. Optimising read/write speeds in immutable operations while keeping an attractive API is not trivial, and Immutable.js uses advanced techniques ([persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure) and [structural sharing](https://en.wikipedia.org/wiki/Hash_array_mapped_trie)) to achieve just that.

While not providing all the bells and whistles, Timm is 10 times smaller (~ 1 kB minified and compressed) and uses much more basic tools, although it also focuses on performance. Reads are as fast as native (you don't even need the library for that), and writes are also reasonably fast ([benchmarks](https://github.com/guigrpa/timm#benchmarks)):

* On small arrays/objects: same performance as Immutable.js for shallow writes, becoming increasingly faster than Immutable.js for deeper writes.

* On large arrays or objects (see *Array: write* in the figure below, where arrays have 1000 elements): increasingly slower than Immutable.js as arrays become longer or objects have more properties. This is where Immutable.js really shines!

* Faster (even *extremely* faster) than [Seamless-immutable](https://github.com/rtfeldman/seamless-immutable), which also uses native objects.

<a href="https://github.com/guigrpa/timm#benchmarks">
    <img src="{{ site.baseurl }}/img/timm-benchmarks.png" alt="Performance comparison">
</a>
<span class="caption text-muted">Read/write benchmarks for Immutable.js, Timm and Seamless-immutable</span>


## ...and what about the JS spread operator?

Finally, why not exploit ES6's shiny new [array spread operator](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator) and its upcoming [sibling for objects](https://github.com/sebmarkbage/ecmascript-rest-spread)? After all, native syntax can't be beaten! Why not do the following?

```js
const obj = { foo: 'foo', bar: 'bar' };

// Why not do this...
const obj2 = { ...obj, bar: 'boo' };

// ...instead of this?
const obj2 = timm.merge(obj, { bar: 'boo' });
```

The object spread operator certainly seems nicer. However, beware:

```js
const obj = { foo: 'foo', bar: 'bar' };

// Future spread operator without changes
const obj2 = { ...obj, bar: 'bar' };
console.log(obj2 === obj);
// -> false

// Timm without changes
const obj2 = timm.merge(obj, { bar: 'bar' });
console.log(obj2 === obj);
// -> true!
```

The spread operator *always* creates a new object, no matter whether we are actually modifying the original one. By contrast, Timm and Immutable.js only instantiate a new one (lazily) when they detect that the operation is in fact mutating the object; this way, you'll create fewer objects, alleviating memory requirements and garbage collection events. And what's more: you'll get fewer false positives when checking whether an object has changed.


## Conclusion

After all this, which is the most suitable option?

* Don't forget the obvious: do you really need immutability? Probably yes, but use your best judgement. If performance is crucial and you can sacrifice code elegance and the other nice perks immutability brings along, well... just **mutate in peace!** I mean, *in place*.

* If you need a complete, battle-tested library and don't mind the lock-in associated to a non-native API even for reads: use **Immutable.js**.

* If you prefer plain arrays/objects, use **Timm**.

* If your typical use cases involve more reading than writing, use **Timm** as well.

* If you do a lot of writes on very long arrays or fat objects, use **Immutable.js**.
