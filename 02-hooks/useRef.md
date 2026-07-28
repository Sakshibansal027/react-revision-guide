# useRef

## The problem useRef solves

We already know two things:

1. A normal variable (like `let x = 0`) resets on every render. It does
   not survive between renders.
2. `useState` does survive between renders, but every time it is updated
   (using its setter function), it causes the component to re-render.

Sometimes, a value is needed that behaves a bit differently from both of
these:

- It should survive across re-renders, like `useState`.
- But changing it should **not** cause a re-render, unlike `useState`.

This is exactly what `useRef` is for.

---

## What useRef returns

```jsx
const countRef = useRef(0);
```

`useRef(0)` returns a plain object that looks like this:

```js
{ current: 0 }
```

It is just one object, with a single property called `current`, holding
whatever initial value was passed in. This value can be read and changed
freely, and React does not track or react to it changing.

---

## Example: a click counter that does not re-render

```jsx
function ClickTracker() {
  const countRef = useRef(0);

  function handleClick() {
    countRef.current = countRef.current + 1;
    console.log(countRef.current);
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

Tracing through five clicks:

- Click 1: `countRef.current` becomes `1`. Console prints `1`.
- Click 2: becomes `2`. Console prints `2`.
- Click 3, 4, 5: becomes `3`, `4`, `5`. Console prints each one.

The screen does not show this count anywhere, because the JSX never
references `countRef.current` - the button always just says "Click me."
Even if the JSX did include `{countRef.current}`, it still would not
update on screen, because changing `.current` does not cause a
re-render. React has no way of knowing that value changed.

---

## What "re-render" actually means

Re-render does not directly mean "something changed on screen." A render
has two parts:

1. **Render (calculation):** React runs the component function again,
   from top to bottom, to calculate what the UI should look like based
   on the current state and props.
2. **Commit (actual screen update):** React compares the newly calculated
   result to what was there before. If nothing actually changed in the
   output, nothing visually updates on screen, even though the function
   did run.

So it is possible for a component to re-render (the function runs again,
using CPU time) without anything visibly changing on screen, if the
result happens to look the same, or if the changed value is not used
anywhere in the JSX.

### Example: re-render happens, but nothing shows differently

```jsx
function ClickTracker() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // this always causes a re-render
  }

  return <button onClick={handleClick}>Click me</button>;
  // count is never used in the JSX here
}
```

Calling `setCount` always causes React to run this function again (a
re-render), regardless of whether `count` is used in the JSX or not.
Since it is not used here, nothing visibly changes on screen, but the
function still runs completely again every time - which is wasted work,
especially if there is other, heavier code inside the component.

### Why this matters for useRef

```jsx
function ClickTracker() {
  const countRef = useRef(0);

  function handleClick() {
    countRef.current = countRef.current + 1; // no re-render happens at all
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

With `useRef`, changing `.current` does not cause the component function
to run again at all. If there were any other calculations inside this
component, they would not re-run just because of this click, since no
re-render is triggered. This is the real advantage of `useRef` over
`useState` when a value does not need to affect what is shown on screen -
it avoids the CPU cost of re-running the whole component unnecessarily.

**Simple rule to remember:**
- `useState`: changing the value always triggers a re-render, whether or
  not that value is shown in the JSX.
- `useRef`: changing the value never triggers a re-render on its own. It
  is a silent, background value.

---

## The second big use of useRef: referencing DOM elements

```jsx
function TextInput() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

Writing `ref={inputRef}` on the `<input>` tells React to set
`inputRef.current` to point directly at the actual DOM element (the real
`<input>` tag as it exists in the browser), once it has been rendered on
screen. This is called a DOM reference.

`inputRef.current.focus()` then calls the browser's real `.focus()`
method directly on that input element, moving the cursor into it.

`useState` would not fit this use case, because the goal here is not to
remember a value that affects what gets rendered. The goal is to get a
direct handle on the actual DOM element, so that browser methods can be
called on it directly - things like `.focus()`, `.scrollIntoView()`, or
`.play()` for a video element. This has nothing to do with re-rendering
or displaying data in JSX; it is about reaching directly into the real
DOM.

---

## Summary: two common uses of useRef

1. Storing a value that needs to persist across renders, but should not
   cause a re-render when it changes (like a click counter used only for
   internal logic).
2. Getting a direct reference to a DOM element, in order to call browser
   methods on it directly (like focusing an input, or measuring its
   size).