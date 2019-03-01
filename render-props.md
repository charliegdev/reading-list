# Render Props

Render props is a way of enabling code reuse by telling a component how it should render; this is usually done by passing the `render` function as a prop.

## Motivation

Normally if we have some common logic to reuse among different components, we can extract them into a React-agnostic separate file. This approach works well until the common logic involves view. Here is an example: assume we just made a component that trackers a user's mouse position on the screen, and print in on screen:

```javascript
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

That looks nice. Now, how do we modify this component, so we:

1. Reuse the mouse tracking logic, but
1. Customize how that information is displayed on the screen

Right now, anywhere we use `<MouseTracker />` would only satisfy #1, not #2. They'll all track the mouse cursor location (_which is good_), but they all display "Move the mouse and see its coordinate" (_which is bad_).

Before we present the render props, let's attempt to solve this problem using a naive approach to see its flaws. It is vital to go through those steps before rushing into a new technique or pattern, because on top of how to use it, we also need to be very clear about when to use it, and what the trade-offs are.

Learning a new technique is useless or sometimes even harmful if we don't know when to use it.

## Scenario: Cat and Mouse

Imagine this scenario: we want to make a component called `<Cat />`; it accepts a coordinate object and displays a cat icon at that coordinate. The purpose is to display this cat right at the user's cursor.

So `<Cat />` might looks like this:

```javascript
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

Assuming we're only using that mouse tracking logic once, we can simply modify `<MouseTracker >` so make it only works with `<Cat />`:

```javascript
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

In most applications, that would be enough. However, what if we want a component just like `<MouseWithCat />`, but work with more than just `<Cat />`?
