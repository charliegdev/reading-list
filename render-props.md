# Render Props

Render props is a way of enabling code logic reuse by providing a component its `render` function as a prop.

## Motivation

Normally if we have some common logic to reuse among different components, we can extract them into a React-agnostic separate file. This approach works well, until the common logic involves view. Here is an example: assume we just made a component that trackers a user's mouse position on screen, and print in on screen:

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
1. Customize how that information is displayed on screen

Right now, anywhere we use `<MouseTracker />` would only satisfy #1, not #2. They'll all track the mouse cursor location (_which is good_), but they all display "Move the mouse and see its coordinate" (_which is bad_).

Before we go any further, let's think of a scenario that makes this reusability _actually_ useful; we will use a naive approach to solve it first, then see why it's naive. This is important, because often we rush into a technique or pattern too fast fearing being left behind, without being very clear what problem it tries to solve, and what trade-off we have to make.

## Scenario: Cat and Mouse

Imagine this scenario: we want to make a component called `<Cat />`; it accepts a coordiate object and display a cat icon at that coordinate. The purpose is to display this cat right at the user's cursor.

So `<Cat />` looks like this:

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

In most applications, that would be enough. But, what if we want a component just like `<MouseWithCat />`, but work with more than just `<Cat />`?
