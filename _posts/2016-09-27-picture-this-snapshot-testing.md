---
layout:     post
title:      "Picture this: snapshot testing"
subtitle:   "Jest, reloaded"
date:       2016-09-27 12:00:00
author:     "Guillermo Grau"
header-img: "img/05-camera.jpg"
comments:   true
---

**_TL;DR_ &nbsp;&nbsp;&nbsp;If you're not testing your React (Native) components at all, or are relying on insufficient, high-level smoke tests, you're not alone. Writing those unit tests must be one of the less rewarding experiences for a developer. But wait: *snapshots to the rescue!* [Jest](https://facebook.github.io/jest/)'s newest release features snapshot testing – and might well change your mind.**

[Jest](https://facebook.github.io/jest/)'s latest release has caused quite a stir, and understandably so: its [**snapshot testing**](https://facebook.github.io/jest/docs/tutorial-react.html#snapshot-testing) feature alone is something of a mind-bender. If you're like me, maybe you didn't write unit tests for your React (Native) components for a number of valid reasons:

* *"They're hard to write"*
* *"They're brittle"*
* *"They don't bring so much value if you have at least some integration tests"*

Starting with Jest 14, though, you may need to revisit those arguments. Snapshots are trivial to use, they can easily be updated, and they provide full coverage of your components, dynamic behavior included. How do they work? Even that can be explained in just two bullets:

1. The first time your test renders a component, a snapshot is taken and saved to a file.
2. Upon re-run, the new render is diffed against the snapshot. If there's a mismatch, you choose which one to keep (and maybe correct your application).

On top of that, Jest provides a [comprehensive testing framework](https://facebook.github.io/jest/docs/api.html#content), [good](https://twitter.com/mikenikles/status/772234132436885504) [performance](http://facebook.github.io/jest/blog/2016/03/11/javascript-unit-testing-performance.html), top-notch diffs and error context and [lots of CLI goodies](https://twitter.com/dan_abramov/status/771496951858733057). Even [coverage results](https://facebook.github.io/jest/docs/getting-started.html#use-coverage-to-generate-a-code-coverage-report) out of the box! No wonder it got rave reviews.

Haven't tried snapshots yet? Let's give them a spin. The following examples are loosely based on [Mady](https://github.com/guigrpa/mady), an application for software internationalization. Until now, Mady had absolutely no tests covering its React components.


## A *hello-world* component

We'll try snapshots on a very simple component first. Mady's `Header` is a stateless functional component showing the application name and a couple of buttons. Lots of styles to be tested, but not much more.

The full unit test for this component is so short you might think it's a joke:

```js
/* eslint-env jest */
import React from 'react';
import renderer from 'react-test-renderer';
import Header from '../header';

it('renders correctly', () => {
  const tree = renderer.create(<Header />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

Run Jest, and a new snapshot will be created:

```
exports[`renders correctly 1`] = `
<div
  style={
    Object {
      "WebkitFlex": "0 0 2.5em",
      "alignItems": "center",
      "backgroundColor": "#dcd6ff",
      "display": "flex",
      "flex": "0 0 2.5em",
      "flexDirection": "row",
      "padding": "5px 8px"
    }
  }>
  ...
</div>
`;
```

That's all there is to it, and it will cover every tiny little CSS property you may be using. Change a color? Jest will warn you. Nudge the header one pixel down? Warning again. Tell Jest to update the snapshot for you, and you're done.

Note: with the current version of React, [you may need to add this](https://github.com/facebook/react/issues/7386#issuecomment-238091398) at the beginning: `jest.mock('react-dom')`.


## Mocking children components

Now let's test a slightly more complicated component. Mady's `TranslatorRow` renders the original message to be translated, as well as several translations. But the translations themselves, which are editable, are handled by a child `Translation` component.

<a href="{{ site.baseurl }}/img/mady.png">
    <img src="{{ site.baseurl }}/img/mady.png" alt="Mady's components in action">
</a>
<span class="caption text-muted">Mady's `Header` and `TranslatorRow` components in action</span>

We are writing *unit* tests for `TranslatorRow`, so we don't want `Translation`s to be rendered as well (they will have unit tests themselves). But we *do* want to know which props are passed down to them.

Jest's mocking functionality comes in handy for this. Let's use it on `Translation`:

```js
jest.mock('../translation', () => jest.fn((props) =>
  <div dataMockType="Translation" {...props} />
));
```

Notice how we include in the snapshot the props passed down to the mock: `{...props}`. I also include a `dataMockType` attribute, which I found useful for easily identifying mocked components in the snapshot.

Now, one of the actual unit tests may look like this:

```js
import TranslatorRow from '../translatorRow';

const MSG_WITH_TRANSLATIONS = {
  id: 'msgId',
  text: 'A message',
  translations: [
    { id: 'id1', lang: 'es', translation: 'Un mensaje' },
    { id: 'id2', lang: 'ca', translation: 'Un missatge' },
  ],
};

it('renders correctly a message with translations', () => {
  const tree = renderer.create(
    <TranslatorRow
      msg={MSG_WITH_TRANSLATIONS}
      langs={['es', 'ca']}
    />
  ).toJSON();
  expect(tree).toMatchSnapshot();
});
```

If we look at the stored snapshot, we will see that an empty `div` was correctly instantiated for each translation, together with the correct props:

```
...
<div
  dataMockType="Translation"
  lang="es"
  msg={...}
  translation={
    Object {
      "id": "id1",
      "lang": "es",
      "translation": "Un mensaje"
    }
  } />
...
<div
  dataMockType="Translation"
  lang="ca"
  msg={...}
  translation={
    Object {
      "id": "id2",
      "lang": "ca",
      "translation": "Un missatge"
    }
  } />
```


## Snapshotting Relay containers

In the real application, `TranslatorRow` is a Relay container, so it expects to be passed a `relay` prop. Now, the Relay Higher-Order Component (HOC) lives outside our application and we may not even know the full shape of the `relay` prop.

I found it more straightforward to just leave the HOC out of our unit test. For this purpose, our default export in `translatorRow.js` is the wrapped `TranslatorRow`, but we also expose the unwrapped version as a named export.

```js
const fragments = {
  msg: () => Relay.QL`
    fragment on Message {
      id
      text
      ${Translation.getFragment('msg')}
      translations(first: 100000) { edges { node {
        id
        lang
        ${Translation.getFragment('translation')}
      }}}
    }
  `,
};

class TranslatorRow extends React.PureComponent {
  // ...
}

export default Relay.createContainer(TranslatorRow, { fragments });
export { TranslatorRow as _TranslatorRow };  // just for unit tests!
```

The snapshot test will be exactly as before, except that we now need to use the named export and conform to Relay's connection specs:

```js
import { _TranslatorRow as TranslatorRow } from '../translatorRow';

const MSG_WITH_TRANSLATIONS = {
  id: 'msgId',
  text: 'A message',
  translations: { edges: [
    { node: {
      { id: 'id1', lang: 'es', translation: 'Un mensaje' },
    } },
    { node: {
      { id: 'id2', lang: 'ca', translation: 'Un missatge' },
    } },
  ] },
};

// ...
```


## Testing dynamic behavior with snapshots

React components may respond to user interaction in different ways. One component might update its state and re-render, whereas another one might just call a function passed in as a prop.

One thing they have in common, though, is that they need to attach event handlers to DOM elements to be notified of user interactions. And those handlers happen to be accessible in Jest and integrate well with snapshot testing; isn't that awesome?

### Dynamic tests on a self-rendering component

In this example, `TranslatorRow` is hoverable and re-renders on `mouseEnter` and `mouseLeave`:

```js
class TranslatorRow extends React.PureComponent {
  constructor() {
    super();
    this.state = { hovered: false };
  }

  render() {
    return (
      <div
        className={this.state.hovered ? 'hovered' : ''}
        onMouseEnter={() => this.setState({ hovered: true })}
        onMouseLeave={() => this.setState({ hovered: false })}
      >
        {/* ... */}
      </div>
    );
  }
}
```

To test the component's dynamic behavior, Jest gives us access to the full rendered tree, *including* event handlers we can manually trigger:

```js
it('changes its class upon mouseEnter/mouseLeave', () => {
  const component = renderer.create(
    <TranslatorRow
      msg={MSG_WITH_TRANSLATIONS}
      langs={['es', 'ca']}
    />
  );
  let tree = component.toJSON();
  expect(tree).toMatchSnapshot(); // snapshot 1

  tree.props.onMouseEnter();
  tree = component.toJSON();      // re-renders
  expect(tree).toMatchSnapshot(); // snapshot 2

  tree.props.onMouseLeave();
  tree = component.toJSON();      // re-renders
  expect(tree).toMatchSnapshot(); // snapshot 3
});
```

Jest will store not one, but three snapshots for this test, capturing all details of how the component should render in the idle and hovered states.


### Dynamic tests with callback props

In this second example, `TranslatorRow` responds to `click` events by calling its `onClick` prop with its message ID:

```js
class TranslatorRow extends React.PureComponent {
  render() {
    const { onClick, msg } = this.props;
    return (
      <div onClick={() => onClick(msg.id)}>
        {/* ... */}
      </div>
    );
  }
}
```

To test this behavior, we use Jest mock (or *spy*) functions:

```js
it('reacts correctly to click events', () => {
  const spyOnClick = jest.fn();
  const tree = renderer.create(
    <TranslatorRow
      msg={MSG_WITH_TRANSLATIONS}
      langs={['es', 'ca']}
      onClick={spyOnClick}
    />
  ).toJSON();
  tree.props.onClick();
  expect(spyOnClick).toBeCalledWith(MSG_WITH_TRANSLATIONS.id);
});
```

OK, we're not using snapshots at all in this example, but I included it to give you a broader picture.


## Conclusions

This has barely scratched the surface of what [Jest](http://facebook.github.io/jest/) can do. After a period in which it was broadly perceived as hard to use and limited to Facebook's own use cases, [Chris Pojer](https://twitter.com/cpojer) *et al* did a great rewrite. It's not just hype: give it a test drive!

I hope this post gave you some ideas to finally address those virgin areas of your project that have never seen a test. I'll whisper this to your ear: snapshot testing can even... *be fun!*
