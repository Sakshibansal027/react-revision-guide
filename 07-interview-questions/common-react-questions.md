# Common React Interview Questions

A quick-reference list of commonly asked React interview questions,
compiled from all the topics in this guide. Each answer is intentionally
short - refer back to the linked topic file for the full explanation and
examples.

---

## Fundamentals

**Q: What is JSX, and how is it different from HTML?**
JSX looks like HTML, but it compiles down to `React.createElement()`
calls. It is not valid on its own in the browser - it needs a build tool
like Babel to convert it into plain JavaScript.
See: [JSX and Rendering](../01-fundamentals/jsx-and-rendering.md)

**Q: Why can't a component return two sibling elements without a wrapper?**
A JavaScript function can only return one value, and JSX compiles to a
single `React.createElement()` call. A Fragment (`<>...</>`) or a
wrapping element solves this without adding an extra DOM node.
See: [JSX and Rendering](../01-fundamentals/jsx-and-rendering.md)

**Q: Why can't an `if` statement be used directly inside `{}` in JSX?**
JSX curly braces only accept expressions (things that produce a value),
not statements. `if` is a statement, so it cannot be used directly
there. A ternary, `&&`, or an early return outside the JSX solves this.
See: [JSX and Rendering](../01-fundamentals/jsx-and-rendering.md)

**Q: What is the difference between a component and an element?**
An element is a plain object describing what to show on screen - just
data. A component is a function (or class) that returns elements.
See: [Components and Props](../01-fundamentals/components-and-props.md)

**Q: Are props mutable or immutable, and why?**
Props are immutable (read-only). This keeps data flow predictable -
data flows down from parent to child, and if a child needs to change
something, it calls a function passed down by the parent instead of
mutating the prop directly.
See: [Components and Props](../01-fundamentals/components-and-props.md)

**Q: What is the children prop?**
Whatever is placed between a component's opening and closing tags
automatically becomes that component's `children` prop. It is commonly
used to build reusable wrapper or container components.
See: [Components and Props](../01-fundamentals/components-and-props.md)

**Q: What is the difference between PropTypes and TypeScript?**
PropTypes only validates props, and only at runtime, with a console
warning. TypeScript checks the entire codebase at compile time, catching
mismatches before the code even runs.
See: [Default Props and Type Checking](../01-fundamentals/default-props-and-type-checking.md)

---

## Hooks

**Q: Why can't a normal variable be used to store UI state?**
A normal variable resets on every render, since the entire component
function runs again from scratch. Changing it also does not tell React
to re-render, so the screen would never reflect the new value.
See: [useState](../02-hooks/useState.md)

**Q: What does useState return?**
An array with exactly two elements: the current state value, and a
function used to update it.
See: [useState](../02-hooks/useState.md)

**Q: What happens if setCount is called multiple times in a row using the current state value directly?**
All calls in that same function use the same "stale" value from that
render, so they overwrite each other instead of adding up. Passing a
function (`setCount(prev => prev + 1)`) fixes this, since React
guarantees it receives the latest value each time.
See: [useState](../02-hooks/useState.md)

**Q: What is useEffect used for?**
Running side effects - code that is not directly about what to render,
like API calls, timers, event listeners, or subscriptions - after the
component has rendered.
See: [useEffect](../02-hooks/useEffect.md)

**Q: What do the different dependency array options in useEffect do?**
No array: runs after every render. Empty array: runs once, after the
first render only. Array with values: runs after the first render, and
again whenever any listed value changes.
See: [useEffect](../02-hooks/useEffect.md)

**Q: Why does useEffect sometimes need a cleanup function?**
If an effect sets something up (a timer, a listener, a subscription), it
needs a matching cleanup to remove it - otherwise it keeps running in
the background even after the component is gone, wasting resources or
causing errors.
See: [useEffect](../02-hooks/useEffect.md)

**Q: What is the difference between useMemo and useCallback?**
useMemo memoizes a calculated value. useCallback memoizes a function
itself (its reference), so the same function object is reused across
renders instead of a new one being created each time.
See: [useMemo and useCallback](../02-hooks/useMemo-and-useCallback.md)

**Q: Why are functions recreated on every render by default?**
Because they are defined inside the component function body, which runs
again from scratch on every render, the same way any other variable
would be recreated.
See: [useMemo and useCallback](../02-hooks/useMemo-and-useCallback.md)

**Q: What problem does useRef solve that useState doesn't?**
useRef lets a value persist across renders without causing a re-render
when it changes, unlike useState, which always triggers a re-render on
update.
See: [useRef](../02-hooks/useRef.md)

**Q: What are the two main use cases of useRef?**
Storing a value that should persist but not trigger re-renders (like an
internal counter), and getting a direct reference to a DOM element to
call browser methods on it (like `.focus()`).
See: [useRef](../02-hooks/useRef.md)

---

## Component Patterns

**Q: What is the difference between a controlled and an uncontrolled input?**
An uncontrolled input's value lives only in the browser's internal DOM
state - React does not know what was typed. A controlled input uses
`value` and `onChange` so that React state is the single source of truth
for what is displayed.
See: [Controlled vs Uncontrolled](../03-component-patterns/controlled-vs-uncontrolled.md)

**Q: Why is using an array index as a key a bad practice?**
If the list is reordered or an item is removed, the index no longer
reliably identifies the same item across renders. React can incorrectly
reuse internal state (like a checkbox's checked status) for the wrong
item, since it only matches by key, not by actual identity.
See: [Lists and Keys](../03-component-patterns/lists-and-keys.md)

**Q: Why does React need keys in lists at all?**
Keys let React match old elements to new elements across renders, so it
knows which ones are the same (just possibly moved), and which ones are
new or removed. This matters for both performance and correctness.
See: [Lists and Keys](../03-component-patterns/lists-and-keys.md)

**Q: What does optional chaining (?.) do, and why is it useful in JSX?**
If the value before `?.` is `null` or `undefined`, it stops immediately
and returns `undefined`, instead of trying to access a property and
crashing. Useful for safely accessing nested, possibly missing data.
See: [Conditional Rendering](../03-component-patterns/conditional-rendering.md)

**Q: What happens if a component returns null?**
It is completely valid - nothing renders at that spot at all, not even
an empty element. Commonly used for things that should conditionally
disappear entirely, like a badge or a closed modal.
See: [Conditional Rendering](../03-component-patterns/conditional-rendering.md)

**Q: What is a compound component, and why use one instead of a single data prop?**
Multiple components (like `Select` and `Select.Option`) that work
together as a set, typically sharing implicit state through Context.
This is more flexible than passing a single data array, since it allows
actual JSX (icons, custom formatting, nested elements) to be used for
each item, instead of being limited to a predefined data shape.
See: [Compound Components](../03-component-patterns/compound-components.md)

---

## State Management

**Q: What is prop drilling?**
Passing a prop through several component levels that don't actually use
it themselves, just so it can reach a component further down the tree
that does need it.
See: [Prop Drilling and Context API](../04-state-management/context-api-and-prop-drilling.md)

**Q: When should Context be used instead of props?**
When data needs to be shared broadly across the tree and should stay the
same no matter which component reads it, like the current logged-in
user or the current theme. Props are better when each usage of a
component genuinely needs different data, like a button's label.
See: [Prop Drilling and Context API](../04-state-management/context-api-and-prop-drilling.md)

**Q: What problem does Redux solve that Context alone doesn't?**
Context solves sharing data, but not how that data changes. Redux
enforces that all state changes go through defined actions and
reducers, giving a single, traceable place for all update logic, a clear
history of what happened, and finer control over which components
re-render.
See: [Redux vs Context](../04-state-management/redux-vs-context.md)

**Q: What are actions and reducers in Redux?**
An action is a plain object describing what happened (e.g.
`{ type: "ADD_ITEM", payload: item }`). A reducer is a function that
takes the current state and an action, and returns a new state, without
mutating the old one directly.
See: [Redux vs Context](../04-state-management/redux-vs-context.md)

---

## Performance

**Q: What is the Virtual DOM, and why does React use it?**
A lightweight, in-memory representation of the UI, made of plain
JavaScript objects. Comparing two Virtual DOM trees in memory is much
faster than directly comparing or modifying the real, slow browser DOM,
so React uses it to figure out the minimal set of real DOM changes
needed.
See: [Reconciliation](../05-performance/reconciliation.md)

**Q: What is reconciliation?**
The overall process React uses to compare the previous and new Virtual
DOM trees (diffing), and update only the minimal necessary part of the
real DOM, instead of rebuilding everything from scratch.
See: [Reconciliation](../05-performance/reconciliation.md)

**Q: What does React.memo do?**
It wraps a component so that React skips re-rendering it if its props
have not changed (compared using reference equality) since the last
render.
See: [Memoization](../05-performance/memoization.md)

**Q: Why doesn't React.memo work well with object, array, or function props by default?**
Because a new object, array, or function is created on every render by
default, even if its contents look identical. Since React.memo compares
props by reference, it sees these as "different" every time, unless
useMemo or useCallback is used to keep the same reference across
renders.
See: [Memoization](../05-performance/memoization.md)

**Q: What is code splitting, and how does React.lazy help with it?**
Code splitting means breaking one large JavaScript bundle into smaller
chunks, each loaded only when needed, instead of all upfront.
`React.lazy()` combined with dynamic `import()` lets a component's code
be fetched only when it is actually rendered for the first time.
See: [Code Splitting and Lazy Loading](../05-performance/code-splitting-lazy-loading.md)

**Q: What is Suspense used for with lazy loading?**
It provides a fallback UI (like a loading message) to show while a lazy
component's code is still being downloaded, instead of showing a blank
screen or an error during that gap.
See: [Code Splitting and Lazy Loading](../05-performance/code-splitting-lazy-loading.md)

---

## Advanced

**Q: What happens by default if a component throws an error while rendering?**
React unmounts the entire component tree, not just the broken
component, resulting in a blank screen, with the actual error visible
only in the browser console.
See: [Error Boundaries](../06-advanced/error-boundaries.md)

**Q: What is an Error Boundary, and why must it be a class component?**
An Error Boundary catches errors in its child components and displays a
fallback UI instead of crashing the whole app. It must be a class
component because the required lifecycle method,
`getDerivedStateFromError`, currently has no hook-based equivalent for
function components.
See: [Error Boundaries](../06-advanced/error-boundaries.md)

**Q: Why place Error Boundaries around specific sections instead of the whole app?**
Wrapping the whole app in one Error Boundary means any single error
takes down the entire UI's fallback, including unrelated sections. Wrapping
only specific, independent sections limits the impact of a bug to just
that section.
See: [Error Boundaries](../06-advanced/error-boundaries.md)

**Q: What is a Portal, and what problem does it solve?**
A Portal lets a component's actual HTML be rendered in a different part
of the real DOM than where it sits in the React tree, while still
behaving normally for props, state, and context. This solves cases
where a parent's CSS, like `overflow: hidden` or `z-index`, would
otherwise clip or hide something like a modal.
See: [Portals](../06-advanced/portals.md)

**Q: What is the difference between where a Portal "lives" in the React tree vs the real DOM?**
In the React tree, it stays nested exactly where it was written, still
receiving props and context normally. In the real DOM, its HTML is
physically inserted somewhere else entirely, usually directly under
`<body>`, so it is unaffected by a restrictive parent container's
styling.
See: [Portals](../06-advanced/portals.md)