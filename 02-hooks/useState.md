# useState

## What problem does useState solve?

A normal JavaScript variable does not persist across re-renders, and even
if it did, changing it would not tell React to update the screen.

```jsx
function Counter() {
  let count = 0; // a normal variable

  function handleClick() {
    count = count + 1;
    console.log(count);
  }

  return <button onClick={handleClick}>{count}</button>;
  // the number on screen never updates, even though count is changing
}
```

Every time a component re-renders, its function runs again from scratch.
This means `let count = 0` resets back to `0` on every render. Even though
`handleClick` does change `count` in memory for a moment, React has no way
of knowing that value changed, so it never re-renders the component to
show the new number.

**useState solves two problems:**
1. It lets React remember a value between re-renders, instead of resetting
   it every time.
2. It tells React to re-render the component whenever that value changes,
   so the screen actually updates.

---

## What happens with the buggy version above

```jsx
function Counter() {
  let count = 0;

  function handleClick() {
    count = count + 1;
    console.log(count);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>Increase</button>
    </div>
  );
}
```

Clicking the button multiple times will print increasing numbers in the
console (`1`, `2`, `3`...), but the number shown on the screen stays at `0`
forever. The variable changes in memory, but React never re-renders the
component, so the displayed value never changes. If the component
re-renders for some other reason later, `let count = 0` runs again anyway,
resetting it back to `0`.

---

## What useState returns

`useState(initialValue)` returns an array with exactly two elements.

```jsx
const [count, setCount] = useState(0);
```

- The **first element** (`count`) is the current value of the state. It
  starts as the initial value passed in, `0` in this case.
- The **second element** (`setCount`) is a function used to update the
  state. Calling it does two things: it updates the stored value, and it
  tells React to re-render the component with the new value.

This uses array destructuring, so the names can be anything
(`const [x, setX] = useState(0)` also works), but the convention is always
`[value, setValue]`.

---

## Never mutate state directly

```jsx
const [count, setCount] = useState(0);

function handleClick() {
  count = count + 1; // wrong, React will not notice this change
}
```

This does not work, for the same reason as the plain variable example
above. Changing `count` directly does not tell React anything. The correct
way is to always call the setter function.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>Increase</button>
    </div>
  );
}
```

Now clicking the button updates the number on screen, because `setCount`
tells React to re-render with the new value.

**Rules to remember:**
- `count` is the current value, read-only, just a number.
- `setCount(newValue)` is a function call, used to request a state update.
- Never write `count = count + 1`.
- Never write `setCount = something`, this overwrites the function itself.

---

## The stale value gotcha: calling the setter multiple times

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>Increase by 3</button>
    </div>
  );
}
```

If `count` starts at `0` and the button is clicked once, the result is `1`,
not `3`.

**Why:** while `handleClick` is running, React has not re-rendered yet, so
all three lines see the same `count` value from that render, which is `0`.
Each line is really doing `setCount(0 + 1)`, since `count` does not update
in the middle of a single function call. It stays frozen at whatever value
it was when this render started. All three calls end up telling React "set
count to 1," so the final result is `1`.

This is called using a **stale value** — `count` inside `handleClick` is a
snapshot from that render, not a value that updates live as the function
runs.

---

## The fix: functional updates

Instead of passing a value directly to the setter, a function can be
passed instead. React guarantees this function always receives the most
recent, up to date value, even if the setter is called multiple times in a
row.

```jsx
function handleClick() {
  setCount(prev => prev + 1); // prev is 0, becomes 1
  setCount(prev => prev + 1); // prev is 1, becomes 2
  setCount(prev => prev + 1); // prev is 2, becomes 3
}
```

Now each call correctly builds on the previous one, since React passes in
the latest value each time. The final result is `3`, as expected.

**Rule of thumb:**
- Use `setCount(count + 1)` when updating state only once, based on the
  current render's value. This is fine.
- Use `setCount(prev => prev + 1)` whenever the new value depends on the
  previous state, especially when calling the setter multiple times, or
  inside loops, timeouts, or rapid events.

---

## Why this is a common interview question

This tests whether someone understands that state updates are not
instant, and that values inside a function are frozen snapshots from that
render, not live references. It is easy to assume that a state variable
updates immediately after calling its setter, but it does not. This
question separates people who understand how React's rendering model
works from those who have only memorized the syntax of useState.