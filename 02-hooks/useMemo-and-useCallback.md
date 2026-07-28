# useMemo and useCallback

## The problem: components re-run everything on every render

We already know that every time a component re-renders, its entire
function body runs again from scratch, top to bottom. This applies to
every variable, every calculation, and every function defined inside it.

```jsx
function ProductList({ products }) {
  const expensiveProducts = products.filter(p => p.price > 1000);

  return (
    <ul>
      {expensiveProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

Every time `ProductList` re-renders, for any reason at all, this line runs
again:

```jsx
const expensiveProducts = products.filter(p => p.price > 1000);
```

`.filter()` loops through the entire `products` array again and builds a
brand new array again, even if `products` has not changed at all since
the last render.

If `ProductList` re-renders 10 times, and `products` has 10,000 items
that never actually changed, that is 10,000 checks repeated 10 times over
- 100,000 checks total, for a result that would have been exactly the
same each time. This is wasted work, and can slow down the app,
especially with large data or heavy calculations.

---

## What memoization means

Memoization means remembering the result of a calculation, so that next
time it is needed, if the inputs have not changed, the old result is
reused instead of doing the calculation again.

A simple real-life example: if someone calculates `15 x 23 = 345` on
paper, and is asked the same question five minutes later with the same
numbers, they would not redo the multiplication - they would just reuse
the answer they already have. That is the whole idea behind memoization.

---

## useMemo

`useMemo` remembers a calculated value, so the calculation does not run
again unless something it depends on has actually changed.

```jsx
function ProductList({ products }) {
  const expensiveProducts = useMemo(() => {
    return products.filter(p => p.price > 1000);
  }, [products]);

  return (
    <ul>
      {expensiveProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

How this works:

- `useMemo` takes a function (the calculation to run) and a dependency
  array, the same idea as the dependency array in useEffect.
- React runs the calculation once and remembers the result.
- On future re-renders, React checks whether `products` (the dependency)
  has actually changed.
  - If it has **not** changed, React skips running the calculation again,
    and simply reuses the previously remembered result.
  - If it **has** changed, React runs the calculation again and
    remembers the new result instead.

So across 10 re-renders where `products` never changes, `.filter()` only
actually runs once, not 10 times.

### Why the dependency array matters here too

If the dependency array is left empty, like `useMemo(() => {...}, [])`,
React runs the calculation once on the first render, and then always
reuses that same result forever, even if `products` later changes to
something completely different. This is a bug, since `expensiveProducts`
would keep showing the old, stale result, even though the actual data has
changed.

The dependency array must list everything the calculation actually
depends on, so React knows exactly when it needs to recalculate:

```jsx
const result = useMemo(() => {
  return expensiveCalculation(someValue);
}, [someValue]); // must include everything the calculation depends on
```

---

## The same problem, but for functions

Functions defined inside a component are also recreated on every render,
the same way variables are.

```jsx
function Parent() {
  function handleClick() {
    console.log("clicked");
  }

  return <Child onClick={handleClick} />;
}
```

`handleClick` is defined inside `Parent`, so every time `Parent`
re-renders, this line runs again, and in JavaScript, defining a function
creates a brand new function in memory every time, even if the code
inside looks exactly the same.

A simple proof of this, outside of React:

```js
function makeFunction() {
  return function () {
    console.log("hi");
  };
}

const fn1 = makeFunction();
const fn2 = makeFunction();

console.log(fn1 === fn2); // false, these are two different function objects
```

The same thing happens inside `Parent` - every render creates a new
`handleClick` function, even though it behaves identically to the
previous one.

### Why this matters when a child is wrapped in React.memo

```jsx
const Child = React.memo(function Child({ onClick }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click me</button>;
});
```

`React.memo` tells React to skip re-rendering `Child` if its props have
not changed since the last render.

The problem: `onClick` is `handleClick`, and a brand new `handleClick`
function is created on every render of `Parent`. React compares functions
using reference equality (`===`), and two different function objects are
never equal, even if their code looks the same.

So `React.memo` sees a different `onClick` reference every time, treats
it as a changed prop, and re-renders `Child` anyway - which defeats the
whole purpose of wrapping it in `React.memo`.

---

## useCallback

`useCallback` solves this by remembering the function itself, so the same
function reference is reused across renders, instead of creating a new
one every time.

```jsx
function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []); // empty array = same function reference reused across renders

  return <Child onClick={handleClick} />;
}
```

### Important: useCallback controls creation, not execution

`useCallback` does not call `handleClick` on its own, and it does not run
it in a loop. It only controls whether a new function object gets created
on each render. `handleClick` still only actually runs when something
calls it, like a button click.

Here is what actually happens across renders:

**Render 1:**
- `useCallback(() => {...}, [])` runs.
- Since the dependency array is empty, and there is nothing stored yet,
  React creates the function once and stores it.
- `handleClick` is Function Object A.

**Render 2 (Parent re-renders for any reason):**
- `useCallback(() => {...}, [])` runs again, since the whole component
  function runs again on every render.
- Since the dependency array is still empty, and nothing in it changed,
  React does not create a new function. It simply hands back the same
  function it stored before.
- `handleClick` is still Function Object A, not a new one.

**Render 3, 4, 5, and onward:** the same thing keeps happening. React
keeps reusing that original Function Object A, as long as the dependency
array stays the same.

### Compare this to not using useCallback

```jsx
function handleClick() {
  console.log("clicked");
}
```

Without useCallback, every render creates a new function object -
Function Object A, then B, then C, and so on - all different objects in
memory, even though they behave identically.

With `useCallback` in place, `Child` (wrapped in `React.memo`) sees the
exact same `onClick` function reference across renders, so it correctly
skips unnecessary re-renders.

---

## useMemo vs useCallback

- `useMemo` remembers a **calculated value** - the result produced by
  running some function. On future renders, if dependencies have not
  changed, it skips running the calculation again and returns the
  already-stored result.

- `useCallback` remembers the **function itself** - not a value it
  produces, but the function definition. On future renders, if
  dependencies have not changed, it skips creating a new function object
  and returns the same function reference as before.

**Simple way to remember it:**
useMemo caches a value. useCallback caches a function.

In fact, `useCallback(fn, deps)` works almost the same as writing
`useMemo(() => fn, deps)` - the same underlying idea, just useCallback is
built specifically for the common case of caching functions.