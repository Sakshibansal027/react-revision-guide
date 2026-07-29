# Conditional Rendering Patterns

## Using && with different values

```jsx
function UserGreeting({ user }) {
  return (
    <div>
      {user && <h1>Welcome, {user.name}</h1>}
    </div>
  );
}
```

What happens with different values of `user`:

- **`user` is `null`:** `null && anything` stops immediately and returns
  `null`. React renders `null` as nothing, so the `div` stays empty.
- **`user` is `undefined`:** same idea, `undefined && anything` returns
  `undefined`. React also renders this as nothing.
- **`user` is a real object,** like `{ name: "Sakshi" }`: objects are
  treated as true, so the right side runs, and
  `<h1>Welcome, Sakshi</h1>` gets rendered.

This works correctly here because `null` and `undefined` are exactly the
values React knows how to render as nothing. This pattern only causes
problems when the value is `0` or an empty string, since React actually
does render those as visible text (covered in the JSX fundamentals file).

---

## Optional chaining (?.)

```jsx
{user?.address?.city}
```

`?.` means: if the thing before it is `null` or `undefined`, stop right
there and return `undefined`, instead of trying to read the next
property, which would otherwise cause a crash.

### Without optional chaining

```jsx
{user.address.city}
```

If `user` exists, but `user.address` is `undefined` (for example, the
user has not added an address yet), then `user.address.city` tries to
read `.city` off of `undefined`. This crashes the app with an error like
"Cannot read properties of undefined (reading 'city')."

### With optional chaining

```jsx
{user?.address?.city}
```

If `user.address` turns out to be `undefined`, this stops safely and
returns `undefined` instead of crashing. `undefined` renders as nothing
on screen, instead of breaking the whole app.

This is especially useful when working with data that might have
missing or optional nested fields, which is very common with API
responses, forms, or partially filled out user profiles.

---

## Handling many conditions cleanly

```jsx
function OrderStatus({ status }) {
  if (status === "pending") return <p>Order is pending</p>;
  if (status === "shipped") return <p>Order has shipped</p>;
  if (status === "delivered") return <p>Order delivered</p>;
  return <p>Unknown status</p>;
}
```

This works, but it does not scale well. With only three statuses it is
fine, but with ten statuses, this becomes ten almost identical `if`
blocks - repetitive, and harder to maintain. For example, changing the
wrapping `p` element would mean editing it in every single block.

### A cleaner approach: using an object as a lookup table

```jsx
function OrderStatus({ status }) {
  const messages = {
    pending: "Order is pending",
    shipped: "Order has shipped",
    delivered: "Order delivered",
  };

  return <p>{messages[status] || "Unknown status"}</p>;
}
```

Here, `messages[status]` looks up the correct text directly using the
status as a key, with no repeated `if` checks. Adding a new status just
means adding one new line to the object, not a whole new `if` block. This
scales much better as the number of possible statuses grows.

A `switch` statement is another valid way to handle many conditions, but
the object lookup pattern is generally considered cleaner for cases like
this.

---

## Returning null from a component

```jsx
function Warning({ show }) {
  if (!show) return null;
  return <p>Something needs your attention!</p>;
}
```

Returning `null` from a component is completely valid in React. It means
"render nothing here, no DOM element at all at this spot." It is not an
empty `div`, and it does not cause an error - genuinely nothing gets
added to the page at that location.

This is a common, intentional pattern for components that need to
completely disappear under certain conditions, such as a warning banner,
a closed modal, or a badge that should only show up sometimes.

### Example: a notification badge

```jsx
function Badge({ count }) {
  if (count === 0) return null;
  return <span>{count} new</span>;
}
```

- If `count` is `0`: the condition `count === 0` is true, so the
  component returns `null`, and nothing renders on screen at all.
- If `count` is `5`: the condition is false, so it skips past
  `return null`, and renders `<span>5 new</span>`, showing "5 new" on
  screen.

This is a clean pattern for badges, notification counts, and warning
banners - anything that should completely disappear when there is
nothing to show, instead of displaying an empty or awkward looking
element.