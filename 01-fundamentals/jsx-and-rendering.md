# JSX and Rendering

## What is JSX, really?

JSX looks like HTML, but it is not HTML. It is a special syntax that gets
converted into plain JavaScript before it runs in the browser. A tool called
Babel does this conversion.

```jsx
// This JSX...
const element = <h1>Hello</h1>;

// ...gets converted into this:
const element = React.createElement('h1', null, 'Hello');
```

**Simple way to say it:**
JSX lets us write UI in a format that looks like HTML, but it is actually
converted into `React.createElement()` calls behind the scenes.

---

## Why can't you return two elements next to each other?

A JavaScript function can only return one value. Since JSX gets converted
into a `React.createElement()` call, and that returns a single object, this
does not work:

```jsx
// Not allowed — returning two elements directly
return (
  <h1>Hello</h1>
  <p>Welcome</p>
);
```

**Fix:** wrap them inside one parent element. You can use a `<div>`, or use
a Fragment (`<>...</>`), which groups elements together without adding an
extra element to the actual page.

```jsx
// Correct
return (
  <>
    <h1>Hello</h1>
    <p>Welcome</p>
  </>
);
```

---

## Code written after return does not run

Once a function hits a `return` statement, it stops running immediately.
Anything written after `return` is never executed.

```jsx
// console.log will never run here
function Greeting() {
  return (
    <div>
      <h1>Hello</h1>
    </div>
  );
  console.log("rendered");
}
```

```jsx
// Correct — move it before return
function Greeting() {
  console.log("rendered");
  return (
    <div>
      <h1>Hello</h1>
    </div>
  );
}
```

**Why this is asked in interviews:** it looks like a React-specific issue,
but it is actually basic JavaScript behavior. This question checks if
someone understands how JavaScript works underneath React, instead of just
memorizing React patterns.

---

## Expressions vs statements (and why it matters in JSX)

- An **expression** produces a value. It can be placed after an `=` sign, or
  passed into a function. Examples: `5 + 3`, `name`, `isActive ? "Yes" : "No"`.
- A **statement** is an instruction that performs an action, but does not
  produce a value by itself. Examples: `if (...) {...}`, `for (...) {...}`.

**JSX curly braces `{}` only allow expressions. They do not allow statements.**

```jsx
// Not allowed — if is a statement, cannot be used inside {}
<div>
  {if (name) {
    return <h1>Hello, {name}</h1>;
  }}
</div>
```

This is why a plain `if` block does not work inside JSX. Instead, we use
patterns based on expressions, shown below.

---

## Conditional rendering: three patterns to know

```jsx
// 1. Ternary operator — use when there is an "else" case
<div>
  {name ? <h1>Hello, {name}</h1> : <h1>Hello, Guest</h1>}
</div>

// 2. Logical && — use only when there is no "else" case
<div>
  {name && <h1>Hello, {name}</h1>}
</div>

// 3. Early return — cleanest option for larger conditional logic
function Welcome({ name }) {
  if (!name) {
    return <h1>Hello, Guest</h1>;
  }
  return <h1>Hello, {name}</h1>;
}
```

The early return version works because the `if` statement is written in
plain JavaScript, outside of JSX. The rule against `if` only applies inside
the `{}` of JSX.

---

## Common bug: && rendering a 0 on screen

```jsx
function Cart({ items }) {
  return (
    <div>
      {items.length && <p>{items.length} items in cart</p>}
    </div>
  );
}
// items = []
```

When `items.length` is `0`, the expression `0 && <p>...</p>` returns `0`
itself, not `false`.

React does not render `null`, `undefined`, `true`, or `false` on the page.
But React does render numbers, including `0`. So when the cart is empty,
the number `0` shows up on the screen instead of nothing.

**Fix:** make sure the condition is a real boolean, not a number.

```jsx
// Correct
{items.length > 0 && <p>{items.length} items in cart</p>}
```

**Rule to remember:** never use a raw number directly with `&&`. Use
`length > 0` or `!!length` instead.