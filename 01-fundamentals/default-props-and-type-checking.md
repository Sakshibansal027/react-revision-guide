# Default Props and Type Checking

## What happens if a prop is not passed?

If a parent does not pass a prop that a child expects, React does not throw
an error. The prop simply becomes `undefined` inside the child, the same
way any missing property on a JavaScript object would be `undefined`.

```jsx
function Counter({ count }) {
  return <p>Count: {count}</p>;
}

<Counter />
// count is undefined here — no error, it just renders "Count: undefined"
```

This is exactly why default values matter. Without one, the screen would
literally show "undefined" to the user, which looks broken.

---

## Setting default values for props

**Option A — destructuring default (most common, cleanest way):**

```jsx
function Counter({ count = 0 }) {
  return <p>Count: {count}</p>;
}
```

This means: if `count` is `undefined`, use `0` instead. This only applies
when the value is missing (`undefined`), and nothing else.

**Option B — fallback using `||` inside the function body:**

```jsx
function Counter({ count }) {
  const safeCount = count || 0;
  return <p>Count: {safeCount}</p>;
}
```

This works too, but has a gotcha: `||` replaces any falsy value, not just
`undefined`. So if a valid value could genuinely be `0`, `false`, or `""`,
using `||` would wrongly replace it with the default as well. Destructuring
defaults (Option A) are usually preferred because they only apply when the
value is exactly `undefined`.

---

## Why checking prop types matters

JavaScript does not stop you from passing the wrong type of value into a
component. This can cause the app to break at runtime, sometimes in a way
that is hard to trace back to the real cause.

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name.toUpperCase()}</h1>;
}

<Welcome name={42} /> // a number was passed instead of a string
```

Here, `42.toUpperCase()` will crash the app, because numbers do not have a
`toUpperCase()` method. Type checking helps catch this kind of mistake
early, ideally with a clear warning during development, instead of a
confusing crash later.

---

## PropTypes

PropTypes is a library that lets you declare what type each prop should be.
If the wrong type is passed, it shows a **console warning** during
development. It does not stop the code from running.

```jsx
import PropTypes from 'prop-types';

function Welcome({ name }) {
  return <h1>Hello, {name}</h1>;
}

Welcome.propTypes = {
  name: PropTypes.string,
};
```

If `<Welcome name={42} />` is used, a console warning like this appears:
"Failed prop type: name should be a string."

---

## TypeScript

TypeScript is a superset of JavaScript that adds type checking to the
entire codebase, not just props. It checks function arguments, return
values, variables, and props, and it catches mismatches **before the code
even runs**, directly in the code editor.

```tsx
type WelcomeProps = {
  name: string;
};

function Welcome({ name }: WelcomeProps) {
  return <h1>Hello, {name}</h1>;
}
```

If `<Welcome name={42} />` is used here, the editor immediately shows an
error, before the code is even run.

---

## PropTypes vs TypeScript

| | PropTypes | TypeScript |
|---|---|---|
| Scope | Only checks props | Checks the entire codebase |
| When it catches errors | At runtime, while the app is running | At compile time, before the code runs |
| What happens on a mismatch | Shows a console warning | Shows an editor error, may block the build |

**Interview-ready summary:**
PropTypes only validates props, and only at runtime, with a warning.
TypeScript checks types across the whole codebase at compile time,
catching bugs before the code ever runs. This is why most modern companies
prefer TypeScript over PropTypes today.