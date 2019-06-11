---
title: "Intro to React Hooks"
description: "This is a quick intro into hooks. You can easily switch over, although the React team recommends doing this in phases, not all at once. They aren’t deprecating classes probably ever, so no rush…"
date: "2019-02-12T17:02:01.728Z"
categories: 
  - JavaScript
  - React
  - Reactjs

published: true
canonical_link: https://medium.com/joelachance/intro-to-react-hooks-1e6572579a8b
redirect_from:
  - /intro-to-react-hooks-1e6572579a8b
---

![](./asset-1.png)

This is a quick intro into hooks. You can easily switch over, although the React team recommends doing this in phases, [not all at once](https://reactjs.org/docs/hooks-intro.html). They aren’t deprecating classes probably ever, so no rush.

Learning about hooks in the 16.8 release ([I was late to this party, apparently](https://www.infoq.com/news/2018/11/react-conf-hooks)) and reading more about it, I was hooked 😜 Check it out — we can replace this:

<Embed src="https://gist.github.com/joelachance/b0d641da289bf945abff5526e3472fce.js" aspectRatio={0.357} caption="" />

With Hooks! Here we go:

<Embed src="https://gist.github.com/joelachance/42a172dc2859229e1d4e7cb0350d4e7e.js" aspectRatio={0.357} caption="" />

A deeper, more complete example can be found [here](https://reactjs.org/docs/hooks-effect.html) in the React docs. A couple of call-outs here:

1.  `useState`. This can be declared as many times as you’d like. You don’t need a single declaration in your constructor. The parameter passed to it is your state’s default value. The above is the same as `state = { counter: 0}` .
2.  `useEffect`. Think of this as our lifecycle methods. Again, we can call this method as many times as we’d like inside of our function component. The benefit of this is we can separate concerns this way — before you probably had multiple concerns in each lifecycle method, which gets a little messy (updating state, making a new API call, etc.). The return function of `useEffect` is the same as our `componentDidUnmount` method.
3.  Last, check out our `onClick` function inside of our JSX. You no longer need to call setState to update state. React exposes our state as a method to update. Much simpler and cleaner!

Last note — React hooks help us with code reuse. We no longer need to re-write code in each component that needs to see this counter. We can refactor our stateful logic into other modules to be imported and used later. We could write our `useEffect` like this:

```
import { callAPI } from './api';

const App = () => {

useEffect(callAPI, []); 

// We give useEffect an empty array if we don't return a callback.

...
```

Now, we can test our `callAPI` function, and we have a better, cleaner, more robust codebase. 🎉
