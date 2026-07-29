# Prop Drilling and the Context API

## The problem: passing data through components that don't need it

```jsx
function App() {
  const [user, setUser] = useState({ name: "Sakshi" });
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserBadge user={user} />;
}

function UserBadge({ user }) {
  return <p>{user.name}</p>;
}
```

`Layout` and `Sidebar` receive `user` as a prop, but never actually use
it themselves - they only pass it straight along to their child. They
act as middlemen, forced to know about and forward data they don't even
care about.

With three levels, this is only mildly annoying. But imagine ten levels
deep - every single component in that chain would need to accept `user`
as a prop, just to pass it further down, even though only the component
at the very bottom actually needs it. Renaming `user` to something else
would mean updating all ten components, even though nine of them never
used the value at all.

---

## Prop drilling

This problem is called **prop drilling** - a prop has to be passed
("drilled") through every intermediate level of the component tree,
whether or not each level actually needs it, just so it can reach the
component that does.

---

## Fixing it with Context

```jsx
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: "Sakshi" });
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

function Layout() {
  return <Sidebar />;
}

function Sidebar() {
  return <UserBadge />;
}

function UserBadge() {
  const user = useContext(UserContext);
  return <p>{user.name}</p>;
}
```

Now `Layout` and `Sidebar` don't mention `user` at all - they go back to
being simple components that just render their children. Only
`UserBadge`, the component that actually needs `user`, calls
`useContext(UserContext)` to read it directly, skipping every middleman
component in between.

**Why this is better:** the components in between no longer need to
know about data they don't use. This makes the codebase easier to
maintain - renaming `user` would only mean touching `App` (where it is
provided) and `UserBadge` (where it is used), not every component in
between.

---

## Should Context replace props entirely?

No. Context and props are suited to different situations.

**Context works best for data that is shared broadly across the tree**,
and stays the same no matter which specific component is asking for it.
Examples: the current logged-in user, the current theme (dark or light
mode), language or locale settings, authentication state.

**Regular props are still the right choice for reusable, self-contained
components**, where each usage genuinely needs different data.

```jsx
<Button label="Save" onClick={handleSave} color="blue" />
<Button label="Cancel" onClick={handleCancel} color="gray" />
```

Every usage of `Button` here needs different data - different labels,
different click handlers, different colors. This is exactly what props
are meant for. Trying to force this through Context would not make
sense, since Context is meant for one shared value used by many
consumers, not for uniquely customizing each individual instance of a
component.

---

## Example: dark mode toggle used in multiple places

Imagine a `ThemeToggle` component used in five different places across
an app, and all five need to reflect the same dark mode state at the
same time.

### Using props (prop drilling happens here)

```jsx
<Layout isDarkMode={isDarkMode}>
  <Sidebar isDarkMode={isDarkMode}>
    <Header isDarkMode={isDarkMode}>
      <ThemeToggle isDarkMode={isDarkMode} />
    </Header>
  </Sidebar>
</Layout>
```

`isDarkMode` would need to be drilled through every level of the tree to
reach all five places `ThemeToggle` is used, even through components
that have nothing to do with theming.

### Using Context

```jsx
function ThemeToggle() {
  const { isDarkMode, toggleTheme } = useContext(ThemeContext);
  return (
    <button onClick={toggleTheme}>
      {isDarkMode ? "Dark" : "Light"}
    </button>
  );
}
```

Every `ThemeToggle`, no matter where it sits in the tree, reads directly
from `ThemeContext` - no drilling, and no middleman components forced to
know about theme.

---

## A simple test to decide between props and Context

Ask: if this value changes, should **all usages update together**, or
does each usage have its **own independent value**?

- If all usages should update together (theme, logged-in user,
  language) → use **Context**.
- If each usage genuinely needs its own separate data (a button's
  label, a card's title) → use **props**.