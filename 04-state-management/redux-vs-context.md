# Redux vs Context

## Starting point: a shopping cart shared across many components

Imagine an e-commerce app, where the cart (list of items added) needs to
be read and updated by many different, unrelated components:
`ProductPage` (add to cart), `Navbar` (cart icon with item count),
`CartPage` (full cart list), `CheckoutButton` (needs cart total).

### Using Context alone

```jsx
const CartContext = createContext();

function App() {
  const [cart, setCart] = useState([]);
  return (
    <CartContext.Provider value={{ cart, setCart }}>
      <Navbar />
      <ProductPage />
      <CartPage />
    </CartContext.Provider>
  );
}
```

This works fine at first. There is no prop drilling, and every component
can read `cart` directly using `useContext(CartContext)`.

### Where this gets messy as the app grows

Imagine five different components can update the cart: `ProductPage`
(add item), `CartPage` (remove item, update quantity), `PromoCodeBox`
(apply discount), `CheckoutPage` (clear cart after order). Each one
directly calls `setCart(...)` with its own logic:

```jsx
setCart([...cart, newItem]);            // in ProductPage
setCart(cart.filter(i => i.id !== id)); // in CartPage
setCart([]);                            // in CheckoutPage
```

**The problem:** there is no single, enforced pattern for how the cart
changes. Every component writes its own logic for modifying `cart`,
however it wants. If the cart total ends up wrong somewhere, every single
component that touches `setCart` needs to be checked, since any of them
could be the cause. There is no central record of what changes happened
to the cart, and in what order.

Also, every time `cart` changes, every component reading `CartContext`
re-renders - even components that don't care about that specific change.
For example, `Navbar`, which only needs the item count, would re-render
even if a change to something unrelated (like a promo code) was bundled
into the same context value.

This is the kind of problem Redux is designed to prevent.

---

## Fixing it with Redux

In Redux, no component is allowed to change `cart` directly. Instead,
every possible way the cart can change is defined upfront, in one place,
called a **reducer**.

```js
function cartReducer(state = [], action) {
  switch (action.type) {
    case "ADD_ITEM":
      return [...state, action.payload];

    case "REMOVE_ITEM":
      return state.filter(item => item.id !== action.payload.id);

    case "CLEAR_CART":
      return [];

    default:
      return state;
  }
}
```

This one function is the only place in the entire app where cart logic
is written. Every possible way the cart can change is listed here,
clearly, in one place.

### Dispatching actions instead of setting state directly

Instead of calling `setCart(...)` directly, components send a message
describing what happened, called an **action**:

```jsx
// In ProductPage - adding an item
dispatch({ type: "ADD_ITEM", payload: newItem });

// In CartPage - removing an item
dispatch({ type: "REMOVE_ITEM", payload: { id: 5 } });

// In CheckoutPage - clearing the cart
dispatch({ type: "CLEAR_CART" });
```

A component does not touch `cart` itself - it sends a message saying
"this happened." Redux takes that message, runs it through
`cartReducer`, and the reducer decides exactly how `cart` should change,
following the rules already written in that one function.

---

## Why this is better at a larger scale

### 1. One place to look for bugs

If the cart total is wrong, only `cartReducer` needs to be checked,
since that is the only place where cart logic exists. There is no need
to search through five different components.

### 2. A clear, replayable history of what happened

Since every change goes through a named action (`ADD_ITEM`,
`REMOVE_ITEM`, `CLEAR_CART`), Redux DevTools can show an exact timeline
of actions in order, along with the state at each step. Past states can
even be replayed to see exactly what the cart looked like at any given
point. This is very difficult to do cleanly with plain Context and
`setCart`.

### 3. Only relevant components re-render

Redux (using tools like `useSelector`) lets a component say "only
re-render me if this specific piece of state changes." So `Navbar`,
which only needs the item count, will not re-render just because a
promo code changed something unrelated in the same store.

### 4. Fewer conflicts between developers working on the same app

If cart logic was scattered across five different components, two
developers working on the app at the same time might:

- Write conflicting logic (one assumes cart items have a `discount`
  field, the other does not set it).
- Duplicate work, writing similar cart-total logic slightly differently
  in two places.
- Introduce bugs that are hard to trace, since the cart-modifying code
  is spread across files neither developer fully sees.

With all cart logic living in one `cartReducer` file:

- Both developers know exactly where to add their logic.
- They can see each other's existing logic in the same file before
  adding their own, reducing the chance of conflicting or duplicate
  code.
- Code reviews become simpler too, since a reviewer only needs to check
  one file to confirm the cart is being updated correctly.

---

## Simple way to remember the core difference

- **Context** is a pipe that carries data to wherever it is needed. It
  solves sharing data, but not how updates to that data happen.
- **Redux** is a strict rulebook combined with a pipe. Data is shared,
  and every change to it must go through a defined, traceable process.

---

## When to actually use which

- **Small to medium apps**, or a few pieces of shared state (like theme
  or the current logged-in user) → **Context is usually enough**, and
  adds far less complexity.
- **Large apps**, with a lot of complex, frequently changing,
  interconnected state, especially with multiple developers working on
  it → **Redux** (or similar tools like Zustand or Redux Toolkit)
  becomes genuinely worth the extra structure.