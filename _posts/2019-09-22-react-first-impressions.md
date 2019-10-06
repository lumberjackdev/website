---
layout: post
title: "Learning React: First Impressions"
tags: [learning, javascript, react]
summary: Reacting to React
---

Recently my adventure into learning new technologies brought me into the world of [React](https://reactjs.org/). For those who know me, this is way outside of my technology comfort zone. I haven't touched a frontend JavaScript framework since AngularJS (no, that's not a typo. It was way before the JS was dropped from the name). Most of my impressions are based on the [tutorial](https://reactjs.org/tutorial/tutorial.html) and reading a few blog posts, so please bear with me if I make a few incorrect assumptions. 

### New JavaScript Features
As I mentioned earlier, it's been awhile since I last explored the JavaScript ecosystem. So one of the first things I noticed was that there are a lot of new features in the language. At this point, it felt fairly similar to learning a new language that just had similar syntax to one I already know.

One of the new features I really enjoyed was the arrow function syntax. If you've been living under a rock like me, then here's an example so you know what I mean:
<pre><code class="language-javascript">// Traditional function syntax
elements.map(function(element) {
  return element.length;
});

// New arrow function syntax
elements.map(element => element.length);
</code></pre>
I really appreciate arrow functions because they're more concise and familiar. They also remind me of anonymous functions (or lambdas) in languages like Kotlin or in Java which makes me feel more at home.  

With all of the features I noticed, I eventually went down the path of investigating the different versions of JavaScript. It quickly became apparent that to someone relatively new to the ecosystem that it can be pretty confusing. Luckily I came across an [article](https://benmccormick.org/2015/09/14/es5-es6-es2016-es-next-whats-going-on-with-javascript-versioning) that explains the versioning. If you're as confused as I was, then I highly recommend giving it a read. 

### Components
This was a post about React though, not about how new JavaScript feels. So let's talk about [React Components](https://reactjs.org/docs/react-component.html). From the tutorial and other examples I've seen, I got the impression that Components are the biggest part of React. It seems like most of the time writing a React application will be spent creating custom Components. They are the building blocks of React.

Components are such a big part of React because they essentially let you create custom html elements in your application. React also provides [lifecycle methods](https://reactjs.org/docs/react-component.html#commonly-used-lifecycle-methods) that allow you to hook into Component's lifecycle to perform custom actions as state changes. Two key pieces of Components that stood out to me are JSX and State.

### JSX
I can't talk about Components without getting into [JSX](https://reactjs.org/docs/introducing-jsx.html). When I first saw JSX in the `render()` function of a Component I was admittedly not a fan. I had unpleasant flashbacks to the old JSPs days with Java. But the more I used them and iterated my way through the tutorial, the more they started to feel natural. My initial JSP comparison definitely proved incorrect (phew!). I appreciate how Components allow you to tie together state and display together in a way that makes sense.  

Here's an example `render()` function to illustrate JSX:
<pre><code class="language-jsx">render() {
    return (
      <button
        className="square"
        onClick={() => this.props.onClick()}
      >
        {this.props.value}
      </button>
    );
  }</code></pre>
As you can see you have classic html elements (you can also reference other components) embedded with JavaScript (denoted inside of `{}` blocks). It makes it pretty easy to construct multiple objects using loops or other array functions like `map`.

### State
Managing application state isn't easy. It can pretty confusing actually. I won't get into the problem too much, but [here's](https://thoughtbot.com/blog/the-problem-of-state) a short read on the problem. 

In React, every Component has its own state that's either managed by itself or its parent Component. Essentially if you have state than needs to be seen in two or more separate Components, you need to elevate the state to the parent Component. I have a feeling that with larger applications this may have complications, for now though I'm willing to accept this as a straightforward and convenient solution. 

### What's Next
All in all, I had fun beginning my way back into frontend development. As I worked through the tutorial I realized I was having fun and wanted to go more in depth. That's as a sure sign for me that I'm going to enjoy React. It's a similar feeling when I'm learning any technology that I enjoy, so there will definitely be future posts about it.

One of the important parts of React that I didn't get into was [Hooks](https://reactjs.org/docs/hooks-intro.html). I feel like there's so much to talk about with Hooks, that I'll probably write a follow on post about them. 

I talked briefly about some of the new JavaScript features I enjoyed, so I think it's worth mentioning that I'll most likely go into TypeScript from here. I have a hunch that the type safety and other features of the language will feel more like home for me. 

From here my goal is to get comfortable enough with React that it can become part of my regular toolbox of technologies. I'll be writing an example app or two to help accomplish my goal, and that means writing about the experience as well, so keep an eye out for that.

That's all for now though, thanks for reading!
