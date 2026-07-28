# Components and Props

## Component vs Element

These two terms are often mixed up, but they mean different things.

An **element** is a plain JavaScript object that describes what should
appear on the screen. It is just a description, not the actual thing yet.

```jsx
const element = <h1>Hello</h1>;
// This is really just an object like:
// { type: 'h1', props: { children: 'Hello' } }
```

An element has no methods, no state, and no lifecycle. It is just data.

A **component** is a function (or class) that returns elements. It is the
factory that produces elements. It can accept props, hold state, and decide
what elements to output.

```jsx
function Welcome() {
  return <h1>Hello</h1>; // the component returns an element
}
```

**Simple way to remember it:**
A component is like a recipe. An element is like one plate of food that the
recipe produces.

---

## Props are immutable

Props are read-only. A component that receives props is never allowed to
change them.

```jsx
// Wrong — this tries to change a prop directly
function Welcome(props) {
  props.name = "Guest"; // not allowed
  return <h1>Hello, {props.name}</h1>;
}
```

Even though JavaScript will not throw an error here (objects are mutable by
default in JavaScript), React does not allow this pattern. If you need a
fallback value, create a new variable instead of changing the prop.

```jsx
// Correct
function Welcome(props) {
  const displayName = props.name || "Guest";
  return <h1>Hello, {displayName}</h1>;
}
```

### Why props are designed to be read-only

1. **Predictability** — if a child component could change props on its own,
   the parent would have no way of knowing that its data was changed
   somewhere inside the component tree. This makes bugs very hard to track
   in larger applications.

2. **One-way data flow** — In React, data flows down from parent to child
   through props. If a child needs to change something, it does not change
   the prop directly. Instead, it calls a function that was passed down to
   it by the parent. That function updates the parent's own state, and the
   updated value is then passed back down as a new prop.

3. **Predictable re-renders** — React needs to know exactly when data
   changes so it can update the screen correctly. If props could be changed
   directly, React would not be able to track those changes reliably.

---

## How a child requests a change (without mutating props)

Example: a simple counter, where the state lives in the parent.

```jsx
function App() {
  const [count, setCount] = useState(0); // state lives here, in the parent

  function handleIncrease() {
    setCount(count + 1); // parent updates its own state
  }

  return <Counter count={count} onIncrease={handleIncrease} />;
}

function Counter({ count, onIncrease }) {
  return (
    <div>
      <p>{count}</p>
      <button onClick={onIncrease}>Increase</button>
    </div>
  );
}
```

**Step by step, what happens when the button is clicked:**

1. The user clicks the button inside `Counter` (the child).
2. `Counter` does not change `count` itself. It calls `onIncrease()`, a
   function that was passed to it by the parent.
3. `handleIncrease` actually lives inside `App` (the parent), and it calls
   `setCount(count + 1)`.
4. The state inside `App` changes, so React re-renders `App`.
5. `App` passes the new, updated `count` value back down to `Counter` as a
   fresh prop.
6. `Counter` receives the new prop and displays the updated number.

**Summary line:**
Children cannot modify props directly. Instead, the parent passes a
function down as a prop, and the child calls that function to request a
change. The actual state update always happens in the parent.

---

## The children prop

`children` is a special, automatic prop. Anything placed between a
component's opening and closing tags automatically becomes that
component's `children` prop.

```jsx
<Card title="Profile">
  <p>This is the card content</p>
</Card>
```

Here, `title="Profile"` is a normal, explicitly named prop. The
`<p>This is the card content</p>` part, everything nested inside the tags,
automatically becomes `props.children`. It is the same as writing:

```jsx
<Card title="Profile" children={<p>This is the card content</p>} />
```

but the nested-tag syntax is cleaner and is what is normally used.

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

This component places the `title` inside an `h2`, and renders whatever was
passed inside its tags wherever `{children}` appears.

**Why this is useful:**
It allows building reusable wrapper or container components. A component
like `Card` does not need to know what content goes inside it. It could be
a paragraph, a button, an image, or another component. `Card` only provides
the outer structure, and `children` acts as a placeholder for whatever
content is passed in each time it is used.

Example of reusing the same component with different content:

```jsx
<Card title="Settings">
  <button>Save changes</button>
</Card>
```

This time, `children` is `<button>Save changes</button>`, and the card
renders a "Settings" heading followed by a "Save changes" button.

**Common real-world uses of children:** modal boxes, layout wrappers, card
components, and button wrappers — anywhere a reusable outer shell needs to
hold different content each time it is used.