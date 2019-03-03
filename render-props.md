# Render Props

In this post, I'm going to talk about **render props** in React. This article is based on [React Blog's tutorial on Render Props](https://reactjs.org/docs/render-props.html); I had a hard time understanding that one, so after I grapsed the concept I created my own guide. You could totally read that one first; if it doesn't make much sense, come back to this one, and hopefully I can clear things up for you.

<!-- TOC -->

- [Definition](#definition)
- [Motivation](#motivation)
- [Scenario: Cat and Mouse](#scenario-cat-and-mouse)

<!-- /TOC -->

## Definition

Render props is a way of **reusing a React component by exposing its `render` function as a prop**, like this:

```javascript
<Mouse render={(mouse) => <Cat mouse={mouse} />} />
```

If that definition and example don't make sense yet, don't worry. The rest of this article will guide you step by step, so you'll know:

1. Why kind of problem **render props** intends to solve
1. How to write a component so it uses **render props**
1. What existing libraries already use this pattern

## Motivation

Let's start with a component: in the next file, `<MouseTracker />` trackers a user's mouse position on the screen, and print in on screen:

```javascript
// MouseTracker.jsx
import React, { useState } from 'react';

const MouseTracker = () => {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = ({ clientX, clientY }) => {
    setX(clientX);
    setY(clientY);
  };

  return (
    <div style={{ height: '100vh', width: '100vw' }} onMouseMove={handleMouseMove}>
      <h1>Move the mouse and see its coordinate!</h1>
      <p>The current mouse position is:</p>
      <p>
        x: {x} y: {y}
      </p>
    </div>
  );
};

export default MouseTracker;
```

![rp1](screenshots/render-props/rp1.PNG)

If we inspect our component, we'll see it does 2 things:

1. Track the mouse cursor location to get the value of `x` and `y`. That is our **business logic**; it's related to _how the application works underneath_, not _how the UI is presented to the user_.
1. Display `x` and `y`, and some text on the screen in the `render` function. That is the **view logic**; it dictates how the UI would look like to an end user.

If we

Right now, anywhere we use `<MouseTracker />` would only satisfy #1, not #2. They'll all track the mouse cursor location (_which is good_), but they all display "Move the mouse and see its coordinate" (_which is bad_).

Before we present the render props, let's attempt to solve this problem using a naive approach to see its flaws. It is vital to go through those steps before rushing into a new technique or pattern, because on top of how to use it, we also need to be very clear about when to use it, and what the trade-offs are.

Learning a new technique is useless or sometimes even harmful if we don't know when to use it.

## Scenario: Cat and Mouse

Imagine this scenario: we want to make a component called `<Cat />`; it accepts a coordinate object and displays a cat icon at that coordinate. The purpose is to display this cat right at the user's cursor.

So `<Cat />` might looks like this:

```javascript
// Cat.jsx
import React from 'react';
import PropTypes from 'prop-types';
import cat from './cat.png';

const Cat = ({ mouse }) => <img src={cat} style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />;

Cat.propTypes = {
  mouse: PropTypes.shape({
    x: PropTypes.number,
    y: PropTypes.number
  }).isRequired
};

export default Cat;
```

Assuming we're only using that mouse tracking logic once, we can simply modify `<MouseTracker >` to make it only works with `<Cat />`:

```javascript
// MouseWithCat.jsx
import React, { useState } from 'react';
import Cat from './Cat';

const MouseWithCat = () => {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = ({ clientX, clientY }) => {
    setX(clientX);
    setY(clientY);
  };

  // Instead of displaying an <h1 /> and <p />, just diplay this <Cat />.
  return (
    <div style={{ height: '100vh', width: '100vw' }} onMouseMove={handleMouseMove}>
      <Cat mouse={{ x, y }} />
    </div>
  );
};

export default MouseWithCat;
```

![cat](screenshots/render-props/cat.PNG)

In most applications, that probably is enough: you made a component, `<MouseWithCat />`, which can track mouse location, and is meant to be used with `<Cat />`. However, what if you want better code reuse: we want a component just like `<MouseWithCat />`, but work with more than just `<Cat />`? We naturally reach a good solution: instead of hard coding `<Cat />` as part of the rendered JSX, why not pass it as a prop, like this:

```javascript
// App.jsx
import React, { Component } from 'react';
import MouseWithAnything from './MouseWithAnything';
import Cat from './Cat';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <MouseWithAnything UsedWith={Cat} />
        </header>
      </div>
    );
  }
}

export default App;
```

```javascript
// MouseWithAnything.jsx
import React, { useState } from 'react';
import PropTypes from 'prop-types';

const MouseWithAnything = ({ UsedWith }) => {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = ({ clientX, clientY }) => {
    setX(clientX);
    setY(clientY);
  };

  return (
    <div style={{ height: '100vh', width: '100vw' }} onMouseMove={handleMouseMove}>
      <UsedWith mouse={{ x, y }} />
    </div>
  );
};

MouseWithAnything.propTypes = {
  UsedWith: PropTypes.func.isRequired
};

export default MouseWithAnything;
```
