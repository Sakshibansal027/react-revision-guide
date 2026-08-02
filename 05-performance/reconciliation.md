# Reconciliation

## Why React doesn't rebuild the whole page on every render

Directly changing the real browser DOM is slow. Every time something in
the DOM changes, the browser may need to recalculate the layout (where
everything should be positioned) and then repaint the pixels on screen.
This work happens even for small changes, and doing it repeatedly for
every tiny update would make an app feel slow very quickly.

To avoid this, React keeps a lightweight, in-memory copy of what the UI
should look like, called the **Virtual DOM**. This is just a plain
JavaScript object structure describing the UI, not real browser
elements, so it is much cheaper and faster to create and compare than
actual DOM nodes.

---

## What actually gets updated in the real DOM

```jsx
// Before
<div>
  <h1>Hello</h1>
  <p>Welcome</p>
</div>

// After re-render
<div>
  <h1>Hello</h1>
  <p>Welcome back</p>
</div>
```

React does not rebuild the whole `div`, and does not even rebuild the
whole `p` element. Here is what actually happens:

1. When the component re-renders, React creates a new Virtual DOM tree
   describing what the UI should now look like.
2. React compares this new Virtual DOM tree to the previous one (the one
   from before the update). This comparison process is called
   **diffing**.
3. React finds the exact, minimal difference. In this example, it
   detects that only the text content inside `p` changed, from "Welcome"
   to "Welcome back".
4. React updates only that specific piece of the real DOM - just the
   text node inside `p` - leaving the `div`, the `h1`, and even the `p`
   element itself completely untouched.

This is much faster than replacing entire elements, since touching the
real DOM is the expensive part. React tries to touch as little of it as
possible.

---

## What reconciliation means

**Reconciliation** is the overall process React uses to figure out what
changed between renders, and to update only that specific part of the
real DOM, instead of rebuilding everything from scratch.

This is the same underlying idea already covered with **keys** in lists.
Keys help React match an old list item to a new one, so it knows what to
update, what to move, and what to leave alone, instead of destroying and
recreating every item. Reconciliation is that same matching and diffing
process, applied to the entire UI tree, not just to lists.

---

## The Virtual DOM

The Virtual DOM is React's lightweight, in-memory representation of the
UI - plain JavaScript objects describing what elements should exist, and
what their properties and children are. It is essentially a blueprint,
not the actual thing rendered in the browser.

### Why use it instead of touching the real DOM directly every time

Comparing two plain JavaScript objects (the old Virtual DOM tree and the
new one) is very fast, since it is just in-memory computation. Comparing
and modifying real DOM elements directly is much slower, since the
browser has to do actual rendering work - layout and painting - every
time the real DOM is touched.

React's strategy is:

1. Do the expensive comparison work cheaply, in memory, using the
   Virtual DOM.
2. Only touch the real DOM for the exact, minimal changes that are
   actually needed.

This is why React apps can stay fast even with frequent updates - React
is careful about touching the slow, real browser DOM as little as
possible.

---

## Summary

- **Virtual DOM:** React's lightweight, in-memory copy of the UI, used
  for fast comparisons.
- **Diffing:** comparing the old Virtual DOM tree to the new one, to find
  exactly what changed.
- **Reconciliation:** the overall process of diffing, followed by
  updating only the minimal necessary parts of the real DOM.

**Why this matters:** the Virtual DOM lets React calculate what actually
needs to change using fast, in-memory comparisons, so it only touches the
real, slow browser DOM for the minimal necessary updates, instead of
re-rendering everything on every single change.