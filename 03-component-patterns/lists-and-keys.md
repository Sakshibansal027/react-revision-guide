# Lists and Keys

## The warning React gives

```jsx
function FruitList({ fruits }) {
  return (
    <ul>
      {fruits.map(fruit => <li>{fruit}</li>)}
    </ul>
  );
}
```

Running this shows a console warning: "Each child in a list should have a
unique 'key' prop."

React cares about this because when a list renders, React needs a way to
track which item is which across re-renders, especially when the list
changes - items get added, removed, or reordered. Without keys, React can
only compare list items by their position (the item at index 0, index 1,
and so on), which causes real bugs.

---

## Why using the array index as a key is a problem

```jsx
{fruits.map((fruit, index) => <li key={index}>{fruit}</li>)}
```

This removes the warning, but it does not solve the underlying problem -
it just hides it.

### A concrete example: a checkbox list

Starting state:

```
index 0: apple   [ ] unchecked
index 1: banana  [x] checked   (the user checked this one)
index 2: cherry  [ ] unchecked
```

```jsx
{fruits.map((fruit, index) => (
  <li key={index}>
    <input type="checkbox" />
    {fruit}
  </li>
))}
```

At this point, React has three `<li>` elements, tied to their keys:

```
key=0 -> element with checkbox UNCHECKED
key=1 -> element with checkbox CHECKED
key=2 -> element with checkbox UNCHECKED
```

The checkbox's checked or unchecked state is stored inside that specific
element, tied to its key.

Now "apple" is removed from the front of the list. The new list is
`["banana", "cherry"]`, so on the next render:

```
key=0 -> banana
key=1 -> cherry
```

React compares this to the previous render:

- **key=0:** was "apple" (unchecked), is now "banana". React assumes
  this is the same element as before, just with different text, so it
  reuses the existing element - including its checkbox state, which was
  unchecked. Only the text changes to "banana".

- **key=1:** was "banana" (checked), is now "cherry". Same logic - React
  reuses the existing element, including its checkbox state, which was
  checked. Only the text changes to "cherry".

The result shown on screen:

```
banana   [ ] unchecked   <- wrong, this was never checked before
cherry   [x] checked     <- wrong, this checkbox belonged to banana
```

"Banana," which the user had checked, now shows as unchecked. "Cherry,"
which was never checked, now shows as checked. The checkbox state did not
move together with the correct fruit - it stayed attached to its
position (the key), while the actual fruit at that position changed.

### Why this happens

React assumes that if the key is the same, the element is the same. But
when index is used as the key, the key only reflects position, not the
actual identity of the item. When an item is removed or the list is
reordered, the fruit at each position changes, but the key does not
reflect that - it just reflects "this is index 0, so treat it as the same
element as before."

### Fixing it with proper unique IDs

```jsx
{fruits.map(fruit => <li key={fruit.id}>{fruit.name}</li>)}
```

If "apple" (id: 1) is removed, and "banana" has id: 2, "cherry" has id: 3:

```
id=2 -> banana (checkbox state correctly stays checked)
id=3 -> cherry (checkbox state correctly stays unchecked)
```

Now React correctly recognizes that "banana" is the same banana as
before, since its key (its id) has not changed, and its checkbox state
correctly stays attached to it.

---

## The correct way to assign keys

A key should be:

- **Unique:** no two items in the list share the same key.
- **Stable:** the same real-world item always gets the same key, even if
  the list is reordered, or items are added or removed elsewhere in the
  list.

This usually means using a real ID that already exists in the data, such
as a database ID or another unique identifier:

```jsx
{fruits.map(fruit => <li key={fruit.id}>{fruit.name}</li>)}
```

If the data genuinely has no unique ID, and the list will never be
reordered, filtered, or added to, using the index as a key is technically
safe. As a general rule, it is better to always use a real unique ID when
one is available, since list changes can easily be introduced later
without realizing this bug is being reintroduced.

---

## Why React needs keys at all

When data changes and a component re-renders, React needs to figure out
which elements in the new list are the same as before (possibly just
moved or updated), and which ones are brand new or have been removed.

Keys are how React matches old elements to new elements across renders.
If an element's key stays the same between renders, React treats it as
the same instance, reusing its existing DOM node and internal state, and
only updating what has actually changed. If the key is different, or
missing, React treats it as a completely new element, discarding the old
one and creating a new one from scratch.

This matters both for performance (reusing DOM nodes instead of
destroying and recreating them) and for correctness (preserving internal
component state correctly, as seen in the checkbox example above).