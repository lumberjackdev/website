---
layout: post
title: "Learning React: First Impressions"
tags: [learning, javascript, react]
summary: Reacting to React
published: false
---

Recently my adventure into learning new technologies brought me into the world of [React](https://reactjs.org/). For those who know me, this is way outside of my technology comfort zone. I haven't touched a frontend JavaScript framework since AngularJS (no, that's not a typo. It was way before the JS was dropped from the name). Most of my impressions are based on the [tutorial](https://reactjs.org/tutorial/tutorial.html) and reading a few blog posts. 

### New JavaScript Features
As I mentioned earlier, it's been awhile since I last explored the JavaScript ecosystem. So one of the first things I noticed was that there are a lot of new features in the language. At this point, it felt fairly similar to learning an entirely new language. 

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
This was a post about React though, not about how new JavaScript feels to me. So let's talk about [React Components](https://reactjs.org/docs/react-component.html). From the tutorial and other examples I've seen, I got the impression that Components are the biggest part of React. It seems like most of the time writing application with React will be spent creating custom Components. 

Components are such a big part of React because they essentially let you create custom html elements in your application. React provides [lifecycle methods](https://reactjs.org/docs/react-component.html#commonly-used-lifecycle-methods) that allow you to hook into React's lifecycle for the component in order to perform custom actions. Two key pieces of Components that stood out to me are JSX and State.

### JSX
I can't talk about Components without getting into [JSX](https://reactjs.org/docs/introducing-jsx.html). When I first saw JSX in the `render()` function of a Component I was admittedly not a fan. I had some flashbacks to JSPs with Java which weren't pleasant. But the more I used them and iterated my way through the tutorial, they started to feel more natural. I appreciated Components allow you to tie together state and display together in a way that makes sense.  

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
If you ask anyone, managing application state is hard. Really hard. **Insert a bunch of links in this section**
- Lifting state up
- Immutability seems to be preferred

### What's Next
All in all, I had fun beginning my way back into frontend development. What I noticed is that I worked through the tutorial, I wanted more. I took it as a sure sign for me that I was going to enjoy React. It's a similar feeling when I'm learning any technology that I enjoy. So there will definitely be future posts on React. 

One of the import parts of React that I didn't get into was [Hooks](https://reactjs.org/docs/hooks-intro.html). I feel like there's so much to talk about with Hooks, that I'll probably right a follow on post about them. 

I talked briefly about some of the new JavaScript features I enjoyed, so I think it's worth mentioning that I'll most likely go into TypeScript from here. I have a hunch that the type safety and other features of the language will feel more like home for me. 

From here my goal is to get comfortable enough with React that it can become part of my regular toolbox of technologies. I'll be writing an example app or two to help accomplish my goal, and that means writing about the experience as well.

I'm going to experiment by creating a series of sorts by updating the post with links to follow on posts in the future. That's all for now though, thanks for reading!