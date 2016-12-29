---
layout:     post
title:      "Word documents, <i>the Relay way</i>™"
subtitle:   "Bridging Word templates, GraphQL queries and JavaScript snippets"
date:       2016-12-28 12:00:00
author:     "Guillermo Grau"
header-img: "img/06-bridge.jpg"
comments:   true
---

**_TL;DR_ &nbsp;&nbsp;&nbsp;Sometimes you have to set aside your developer work and do some office stuff. And sometimes that includes generating Word reports with dynamic contents. What if you could bridge Word, GraphQL and JavaScript to help you with these chores?**

Every once in a while you need to produce a Microsoft Word document with dynamic contents. Maybe it's a project report, or the boring economic section of a tender. Chances are you're working within a team, and you absolutely need your document to be in the *de facto* standard for non-nerdy output: Word.

You don't want to write hundreds of tables by hand, so you start thinking about *docx* automation. And since you're a proud Node.js developer, you stumble upon these types of modules from the top of npm's search results:

* *Fully programmatic generators*, like [docx](https://github.com/dolanmiu/docx), which provide functions to create paragraphs, text runs, and so on.

* *Template-based generators*, like [docx-templates](https://github.com/guigrpa/docx-templates), which let you start with an ordinary Word document, adding commands as needed to insert dynamic contents.

The benefits of the second approach are numerous:

1. You don't need the library to support every tiny little feature of Word. The input template will be used transparently, so the output will be compatible with whatever version of Office you're using.

2. You don't need to write code to generate *static* parts of your document.

3. You can even write documents *à la Relay*. What am I talking about? Well, just keep on reading!


## Colocating queries

One of the key ideas in [Relay](https://facebook.github.io/relay/) is *colocation*. It's literally front and center in its home page, and what it basically means is that data requirements (the *query*) are declared very close to the components that need the data.

If we translate this idea to report generation, wouldn't it be nice if we just wrote the data query in the document itself? A very simple template would look like this:

<a href="{{ site.baseurl }}/img/docx-swapi-simple1.png">
    <img src="{{ site.baseurl }}/img/docx-swapi-simple1.png" alt="Query colocation in docx-templates">
</a>
<span class="caption text-muted">Query colocation</span>

We're using the [Star Wars API](https://swapi.co/) with the [GraphQL](http://graphql.org/) [wrapper](https://github.com/graphql/swapi-graphql) for maximum convenience, but `docx-templates` doesn't really care about the language you choose for queries, as long as they eventually resolve to some data.

With the data requirements in place, we can write a short JavaScript snippet to execute the embedded query and create the report.

```js
// swapi.js
const createReport = require('docx-templates');
require('isomorphic-fetch');  // assuming a modern Node, you need no Promise polyfill

createReport({
  template: process.argv[2],
  data: query =>
    fetch('http://graphql-swapi.parseapp.com', {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ query }),
    })
    .then(res => res.json())
    .then(res => res.data),
});
```

Run this script with `node swapi swapi-simple-template.docx` and *voilà*, both trilogies come up in your report (alas, no *Force Awakens* data yet in the GraphQL wrapper 😱):

<a href="{{ site.baseurl }}/img/docx-swapi-simple2.png">
    <img src="{{ site.baseurl }}/img/docx-swapi-simple2.png" alt="A very simple report generated with docx-templates">
</a>
<span class="caption text-muted">A simple report generated with `docx-templates`</span>

## A more complex example

GraphQL is really great for describing complex data requirements. Imagine you want to summarize all Star Wars films with some basic data and a table of characters with their species, home planet, and ships (i.e. vehicles with hyperdrive) they appear in. Really nerdy stuff, huh? No problem with GraphQL:

```
+++QUERY { allFilms { films {
  title, releaseDate, director, producers, openingCrawl
  characterConnection { characters {
    name
    species { name }
    homeworld { name }
    starshipConnection { starships { name crew passengers } }
  } }
} } }+++
```

Your Word template might look like this (right after the query):

<a href="{{ site.baseurl }}/img/docx-swapi-complex2.png">
    <img src="{{ site.baseurl }}/img/docx-swapi-complex2.png" alt="A complete template with embedded JS">
</a>
<span class="caption text-muted">A complete template with embedded JS</span>

This templates demonstrates other features of `docx-templates`:

* You can embed JavaScript in your `INS` (shorthand notation: `=`), `FOR` and other commands.

* You can define aliases (`ALIAS`) for complete commands, which you can then resolve with the `*<alias-name>` shorthand. We found it quite useful when templating tables.

* Loop variables (e.g. `$film`, `$character`) can be used from embedded JavaScript snippets, just like any other property of the query result.

Want to see the generated document? Just run `node swapi swapi-complex-template.docx` and there it is, in all its geeky splendor:

<a href="{{ site.baseurl }}/img/docx-swapi-complex3.png">
    <img src="{{ site.baseurl }}/img/docx-swapi-complex3.png" alt="A complete report generated with docx-templates">
</a>
<span class="caption text-muted">A complete report generated with `docx-templates`</span>

## A final word

[`docx-templates`](https://github.com/guigrpa/docx-templates) is a young project and still evolving with our needs. If you use it and have suggestions or —even better— want to contribute, please visit its GitHub repo!
