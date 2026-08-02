# Memoization: React.memo

This builds on the useMemo and useCallback file. Those two cover
memoizing values and functions - this covers the third piece:
memoizing an entire component, using `React.memo`.

## Children re-render by default, even with unchanged props

```jsx
function Child({ name }) {
  console.log("Child rendered");
  return <p>{name}</p>;
}

function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increase: {count}</button>
      <Child name="Sakshi" />
    </div>
  );
}
```

Every time the button is clicked, `count` changes, and `Parent`
re-renders. `Child` only depends on `name`, which never changes, so
logically it does not need to re-render at all.

**But by default, it will re-render anyway.** When a parent component
re-renders, React re-renders all of its children by default, regardless
of whether their specific props actually changed. So on every click,
`Parent` re-renders because `count` changed, and `Child` re-renders along
with it, even though `name` is exactly the same "Sakshi" as before.
"Child rendered" would be logged in the console on every single click,
which is wasted work.

---

## What React.memo does

```jsx
const Child = React.memo(function Child({ name }) {
  console.log("Child rendered");
  return <p>{name}</p>;
});
```

`React.memo` wraps a component and tells React: before re-rendering this
component, first check if its props actually changed since the last
render. If they are the same, skip re-rendering entirely and reuse the
previous result.

Now, when `Parent` re-renders because `count` changed, React checks
whether `Child`'s props changed. `name` is still `"Sakshi"`, so React
skips re-rendering `Child` completely. "Child rendered" will only log
once, no matter how many times the button is clicked.

---

## Why React.memo needs useCallback for function props

```jsx
const Child = React.memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []); // same function reference every render

  return <Child onClick={handleClick} />;
}
```

`React.memo` checks props using reference equality (`===`). If a
function is passed as a prop without `useCallback`, a brand new function
object gets created on every render, so `React.memo` would always see
"different props" and re-render anyway, defeating its own purpose. This
is why `useCallback` and `React.memo` are typically used together:
`React.memo` skips re-renders when props are the same, and `useCallback`
makes sure function props actually stay the same across renders instead
of being recreated pointlessly.

---

## The same trap applies to objects and arrays

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increase</button>
      <Child user={{ name: "Sakshi" }} />
    </div>
  );
}

const Child = React.memo(function Child({ user }) {
  console.log("Child rendered");
  return <p>{user.name}</p>;
});
```

`{ name: "Sakshi" }` is written directly inside `Parent`'s JSX. Every
time `Parent` re-renders, this line runs again, and it creates a brand
new object in memory each time, even though it looks identical to the
previous one. This is the same situation as functions being recreated on
every render.

`React.memo` compares props using `===`, and two different object
literals are never `===` equal, even with identical content:

```js
{ name: "Sakshi" } === { name: "Sakshi" } // false, always
```

So `React.memo` sees a different `user` object than before, treats it as
changed props, and re-renders `Child` anyway. "Child rendered" would log
on every click, defeating the purpose of wrapping `Child` in
`React.memo`.

### The fix: useMemo to keep the object reference stable

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  const user = useMemo(() => ({ name: "Sakshi" }), []); // same object reference across renders

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increase</button>
      <Child user={user} />
    </div>
  );
}
```

Now `user` is memoized, keeping the same object reference across renders
(since the dependency array is empty), so `React.memo` correctly sees
"same `user` prop as before" and skips re-rendering `Child`.

---

## The key rule to remember

`React.memo` only helps if the props themselves stay referentially
stable across renders.

- For primitives (strings, numbers, booleans), this happens
  automatically - `"Sakshi" === "Sakshi"` is always `true`.
- For objects, arrays, and functions, `useMemo` or `useCallback` is
  needed to keep them stable across renders. Without this,
  `React.memo` silently does nothing useful, since it will always see a
  "new" prop value.

---

## Summary: the three memoization tools together

- `useMemo` memoizes a calculated **value**.
- `useCallback` memoizes a **function**.
- `React.memo` memoizes an entire **component's rendered output**,
  skipping re-render if its props have not changed.

These three tools are usually used together: `React.memo` decides
whether to skip a re-render based on whether props changed, and
`useMemo`/`useCallback` make sure that object, array, and function props
actually stay stable across renders, so that `React.memo` can correctly
detect "nothing changed" instead of always seeing new references.