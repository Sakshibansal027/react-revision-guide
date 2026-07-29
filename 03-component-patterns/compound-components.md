# Compound Components

## The problem with passing everything as one data prop

```jsx
<Select
  options={[
    { label: "Apple", value: "apple" },
    { label: "Banana", value: "banana" },
  ]}
  selected="apple"
  onChange={handleChange}
/>
```

Here, every option is a plain object with a `label` and a `value`. This
works fine for simple cases, but what if one option needs an icon next
to it, another needs to be disabled, and another needs custom styling?

More and more fields would need to be added to each option object -
`icon`, `disabled`, `customStyle`, and so on - and the `Select` component
would need to know how to handle every one of these special cases
internally. This gets messy quickly, and it is hard to predict every
possible customization someone might want in advance.

**The core limitation:** passing data through plain objects or arrays
limits things to whatever shape was planned for in advance. Actual JSX -
elements with their own styling, icons, or nested content - cannot easily
be passed through a plain data prop.

---

## The compound component version

```jsx
<Select value={selected} onChange={handleChange}>
  <Select.Option value="apple">Apple</Select.Option>
  <Select.Option value="banana">Banana</Select.Option>
  <Select.Option value="cherry" disabled>Cherry</Select.Option>
</Select>
```

Instead of passing data, actual JSX children are being written here.
Each `Select.Option` is a full component, so it can contain anything -
icons, custom formatting, nested elements, anything at all. Since it is
just JSX, there is no need for `Select` to predict every possible use
case in advance.

This is more flexible because it uses one of React's core strengths -
composing components together - instead of forcing everything through a
rigid, predefined data structure.

---

## How Select and Select.Option communicate

`Select.Option` needs to know whether it is the currently selected
option, but notice that nobody manually passes `selected="apple"` to
each individual `Select.Option`. So how does it know?

### A simple way to think about it

Imagine a building with a speaker system on the ground floor that
announces something like "current selected value: apple." Every room in
the building can hear this announcement, without anyone needing to walk
around and tell each room individually.

- `Select` is like the speaker making the announcement.
- Each `Select.Option` is like a room that can hear the announcement.

`Select.Option` is not manually told the selected value - it listens to
a shared channel and finds out that way. In React, this shared channel
is called **Context**.

### A simple Context example, without lists

```jsx
const MyChannel = createContext();

function Speaker() {
  return (
    <MyChannel.Provider value="apple">
      <Room />
    </MyChannel.Provider>
  );
}

function Room() {
  const message = useContext(MyChannel); // listening to the channel
  return <p>Heard: {message}</p>; // "Heard: apple"
}
```

Here is what is happening:

1. `createContext()` creates a new channel that anything can broadcast
   on.
2. `<MyChannel.Provider value="apple">` means everything nested inside
   this Provider can access the value `"apple"`, if it chooses to listen.
3. `Room`, which is nested inside the Provider, uses `useContext(MyChannel)`
   to listen to that channel - without `Speaker` ever manually passing
   `Room` a prop directly.

### Applying this to Select and Select.Option

```jsx
const SelectContext = createContext();

function Select({ value, onChange, children }) {
  return (
    <SelectContext.Provider value={{ selectedValue: value, onChange }}>
      <div className="select">{children}</div>
    </SelectContext.Provider>
  );
}

function Option({ value, disabled, children }) {
  const { selectedValue, onChange } = useContext(SelectContext);

  const isSelected = selectedValue === value;

  return (
    <div
      onClick={() => !disabled && onChange(value)}
      style={{
        fontWeight: isSelected ? "bold" : "normal",
        opacity: disabled ? 0.5 : 1,
      }}
    >
      {children}
    </div>
  );
}

Select.Option = Option;
```

Step by step:

1. `Select` wraps its children in a Context Provider, sharing
   `selectedValue` and `onChange` with everything nested inside it.
2. Each `Select.Option` calls `useContext(SelectContext)` to read this
   shared data - this is how it knows whether it is currently selected,
   and what function to call when it is clicked.
3. Whoever uses `<Select>` never has to manually tell each
   `<Select.Option>` what the selected value is - it becomes available
   automatically through Context.

---

## The role of {children}

Whatever content is written between a component's opening and closing
tags automatically becomes that component's `children` prop.

```jsx
<Select value="apple" onChange={handleChange}>
  <Select.Option value="apple">Apple</Select.Option>
  <Select.Option value="banana">Banana</Select.Option>
  <Select.Option value="cherry">Cherry</Select.Option>
</Select>
```

Everything written inside the `Select` tags - the three `Select.Option`
elements - becomes `Select`'s `children`.

```jsx
function Select({ value, onChange, children }) {
  return (
    <SelectContext.Provider value={{ selectedValue: value, onChange }}>
      {children}
    </SelectContext.Provider>
  );
}
```

`{children}` here means: render whatever content was passed inside
`Select`, at this exact spot. If `{children}` was left out, the three
`Select.Option` elements would never render at all - nothing would show
up on screen, since `Select` was never told to display what it was
given.

Both pieces are needed together:

- The Context Provider lets every nested `Select.Option` read the shared
  `selectedValue` and `onChange`.
- `{children}` actually renders those `Select.Option` elements, inside
  that same Provider, so they can access the Context.

Without `{children}`, nothing would render. Without the Context
Provider, things would render, but each `Select.Option` would have no
way of knowing the selected value.

---

## How Select.Option = Option actually works

```js
function Select() { /* ... */ }

Select.Option = function Option() { /* ... */ };
```

In JavaScript, functions are objects, and objects can have properties
attached to them at any time. This line is no different from:

```js
const person = {};
person.name = "Sakshi";
```

Since `Select` is just a function, and functions are objects in
JavaScript, any property can be attached to it - including another
function or component. `Select.Option` is simply JavaScript's normal way
of accessing a property on an object. There is nothing React-specific
about this syntax - it is a naming convention developers use to visually
group related components together.