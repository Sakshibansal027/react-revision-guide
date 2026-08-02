# Error Boundaries

## What happens by default when a component throws an error

React does not show any friendly error message on its own by default.
If a component throws an error while rendering (for example, trying to
read a property on `undefined`), React's default behavior is to unmount
the entire component tree - not just the broken component, but
everything, including completely unrelated parts of the app. The user
sees a blank white screen, and the actual error is only visible in the
browser console, which most users never check.

### Example of the problem

```jsx
function App() {
  return (
    <div>
      <Navbar />
      <UserProfile user={undefined} /> {/* crashes reading user.name */}
      <Footer />
    </div>
  );
}
```

If `UserProfile` crashes here, by default the entire app disappears -
`Navbar` and `Footer` vanish too, even though they had nothing to do
with the bug. One small bug in one component takes down the whole page.

---

## What class components are

All components covered so far have been function components:

```jsx
function Welcome() {
  return <h1>Hello</h1>;
}
```

React also has an older way of writing components, called class
components, which use JavaScript's `class` syntax:

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello</h1>;
  }
}
```

Both do the same job - displaying UI - but they are written differently.
In a class component:

- `class ComponentName extends React.Component` is used, which extends
  a special React class to gain React's features.
- A `render()` method is used to return JSX, instead of a plain
  function's `return`.
- State is stored on `this.state`, instead of using `useState`.

Before hooks were introduced in 2019, class components were the only way
to handle state and lifecycle logic in React. Hooks made function
components just as powerful, without needing classes, which is why
almost all new React code today is written using function components.

### Why class components are still relevant

A few very specific cases still require class components, because React
has not built a hook-based (function component) alternative for them.
**Error Boundaries are exactly one of these cases.**

---

## How Error Boundaries work

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <p>Something went wrong.</p>;
    }
    return this.props.children;
  }
}
```

- `class ErrorBoundary extends React.Component` makes this a class
  component.
- `state = { hasError: false }` is how state is written in a class
  component.
- `static getDerivedStateFromError()` is a special method that only
  exists in class components. When an error is thrown by something
  inside this component (its children), React automatically calls this
  method, which decides how to update state - here, setting
  `hasError: true`.
- In `render()`, if `hasError` is true, a fallback UI is shown ("Something
  went wrong"). Otherwise, the normal children are rendered.

There is currently no hook (like a `useErrorBoundary`) that provides this
same functionality in a function component. This is one of the few
genuine exceptions to the general rule of writing everything as function
components in modern React.

In practice, most apps write this `ErrorBoundary` class once, keep it in
its own file, and reuse it wherever needed, rather than writing a new
class each time.

---

## Where to place Error Boundaries

### Wrapping the whole app (not ideal)

```jsx
function App() {
  return (
    <ErrorBoundary>
      <Navbar />
      <UserProfile user={undefined} />
      <Footer />
    </ErrorBoundary>
  );
}
```

If `UserProfile` crashes here, the entire `ErrorBoundary` catches it and
shows the fallback for everything inside it - so `Navbar` and `Footer`
disappear too, replaced with "Something went wrong." This just recreates
the same problem as before (one bug taking down unrelated parts of the
page), with a nicer message instead of a blank screen.

### Wrapping only the risky section (better)

```jsx
function App() {
  return (
    <div>
      <Navbar />
      <ErrorBoundary>
        <UserProfile user={undefined} />
      </ErrorBoundary>
      <Footer />
    </div>
  );
}
```

Now if `UserProfile` crashes, only that section shows the fallback -
`Navbar` and `Footer` stay fully functional and visible. This contains
the impact of any bug to just the specific part of the UI that actually
broke.

**Common real-world pattern:** wrapping Error Boundaries around
independent, self-contained sections, such as each widget on a
dashboard, or each major feature block, so that if one specific feature
breaks, the rest of the app keeps working normally.