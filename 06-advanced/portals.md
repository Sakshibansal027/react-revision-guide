# Portals

## The problem: modals rendered deep inside nested components

```jsx
function Widget() {
  return (
    <div style={{ overflow: "hidden", position: "relative" }}>
      <p>Some widget content</p>
      <Modal /> {/* rendered here, nested inside Widget's div */}
    </div>
  );
}
```

Since `Modal` is rendered inside `Widget`'s `div`, following the normal
component tree, it is also physically placed inside that `div` in the
actual HTML/DOM. If `Widget`'s container has CSS like `overflow: hidden`
(commonly used to prevent content from spilling out of a box), anything
that visually extends beyond that container's boundaries gets clipped or
cut off - including a modal, even though the modal is meant to appear as
a full-screen overlay on top of everything else.

Similarly, `z-index` (which controls what appears visually on top) only
works properly within the same stacking context. If a parent container
has a lower `z-index` or creates its own separate stacking context, the
modal could end up appearing behind other elements, even with a very
high `z-index` value set on it directly.

---

## What a Portal does

```jsx
import { createPortal } from "react-dom";

function Modal({ children }) {
  return createPortal(
    <div className="modal-overlay">{children}</div>,
    document.getElementById("modal-root") // a different DOM node, usually directly under <body>
  );
}
```

`createPortal` allows a component's actual HTML to be rendered somewhere
else entirely in the real DOM, even though it stays logically nested
inside the component tree for props, state, and React logic purposes.
This target is usually a separate element placed directly under
`<body>`, completely outside of any restrictive parent container like
`Widget`'s `overflow: hidden` box.

---

## The key idea: React tree vs real DOM

Normally, where a component sits in the React component tree and where
its HTML actually appears in the DOM are always the same place. A
Portal creates a split between these two things:

- **In the React tree:** the component still appears nested exactly
  where it was written - it still receives props and context normally
  from its parent, and click events still bubble up to the parent as if
  it were nested there.
- **In the real DOM:** the component's actual HTML elements get inserted
  somewhere else entirely - typically outside the restrictive parent's
  container, directly under `<body>`. Because of this, it is not
  affected by that parent's `overflow: hidden` or `z-index` issues at
  all, since it is no longer physically inside that container.

This is the core mechanism Portals use to solve the visual escaping
problem - keeping the component logically nested for React's purposes,
while physically placing its rendered output somewhere unaffected by
restrictive parent styling.

---

## Common use cases for Portals

- Modals and popups
- Tooltips
- Dropdown menus
- Notification toasts

Anything that needs to visually escape its parent container and appear
on top of the entire page, regardless of where it is logically used in
the component tree, is a good candidate for a Portal.