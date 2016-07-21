---
layout: post
title:  "Using SigmaJS With React"
date:   2016-07-21 09:00:00
categories: blog
---

SigmaJS is a library that allows for graph visualization. You can think of it as a library that is similar to d3.js except it is specific to just graph visualization, is lighter, and allows for WebGL rendering.

<u>*Sigma's Slight Issue With React/Webpack*</u>
---

If you are using Webpack to handle all your npm installations, then you can add Sigma to your project with the following command.

```
npm install sigma
```

Note that there is an existing problem with this build -- [check out this Github issue](https://github.com/jacomyal/sigma.js/pull/653). You should either:

1) Fix it on your end, and then install that into your project, or
2) Install sigma into the project and then do the appropriate changes.

Below are the changes that need to be done:

Original:

```
// Dirty polyfills to permit sigma usage in node
if (HTMLElement === undefined)
  var HTMLElement = function() {};

if (window === undefined)
  var window = {
    addEventListener: function() {}
  };
```

Change this to:

```
// Dirty polyfills to permit sigma usage in node
if (HTMLElement === undefined)
  HTMLElement = function() {};

if (window === undefined)
  window = {
    addEventListener: function() {}
  };
```

If your package is already built you need to do it in two places:
1) node_modules/sigma/build/sigma.require.js (around line 1700), and
2) node_modules/sigma/src/sigma.export.js

<u> *SigmaJS Basic Example Translated into React* </u>
---

The below code snippet is the translation of the [Basic Sigma Example](https://github.com/jacomyal/sigma.js/blob/master/examples/basic.html) to React.

If you are trying to run this example, name your file BasicGraphExample.js - the same as the name of the component.

```
import React from 'react';
import ReactDOM from 'react-dom';
import Sigma from 'sigma';

const g = { };

const styles = {
  container: {
    top: 0,
    bottom: 0,
    left: 0,
    right: 0,
    position: 'absolute',
  },
};

const nodes = Array.from(Array(100).keys()).map((i) => ({
  id: `n${i}`,
  label: `Node ${i}`,
  x: Math.random(),
  y: Math.random(),
  size: Math.random(),
  color: '#666',
}));

const edges = Array.from(Array(500).keys()).map((i) => ({
  id: `e${i}`,
  source: `n${(Math.random() * 100 | 0)}`,
  target: `n${(Math.random() * 100 | 0)}`,
  size: Math.random(),
  color: '#ccc',
}));

g.nodes = nodes;
g.edges = edges;


class BasicGraphExample extends React.Component {
  componentDidMount() {
    new Sigma({
      graph: g,
      container: ReactDOM.findDOMNode(this.refs.container),
    });
  }

  render() {
    return <div ref="container" style={styles.container}></div>;
  }
}

export default BasicGraphExample;
```

Now you should have an awesome random looking graph! How fun! Time to get cracking on making some cool stuff with the library :)
