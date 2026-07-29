# Controlled vs Uncontrolled Inputs

## Uncontrolled input: the browser is in charge

```jsx
<input type="text" />
```

This input has no `value` prop and no `onChange` handler - nothing
connecting it to React state at all. If someone types into this in a
browser, the typed text lives only in the browser's own internal state,
the same way it would on a plain HTML page with no JavaScript at all.

React has no idea what has been typed into this input. If asked "what is
currently inside this input," React genuinely does not know, since it is
never told. This is called an **uncontrolled component** - the browser
controls what is shown, not React.

---

## Controlled input: React is in charge

```jsx
function NameInput() {
  const [name, setName] = useState("");

  return (
    <input
      type="text"
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

Here:

- `value={name}` forces the input's displayed text to always equal
  whatever `name` currently is in React state.
- `onChange={(e) => setName(e.target.value)}` runs every time the user
  types a character, updating `name` in state to whatever was just
  typed.

### Step by step: what happens when the user types "a"

1. The user presses the "a" key.
2. The browser's `onChange` event fires, and `e.target.value` is `"a"`.
3. `setName("a")` runs. This updates React's state and triggers a
   re-render of `NameInput`.
4. `NameInput` runs again from the top. `name` is now `"a"`.
5. The JSX evaluates `value={name}`, which becomes `value="a"`.
6. React sets the actual input element's value to `"a"`. Since this
   matches what the user just typed, nothing looks different on screen -
   it simply shows `a`, as expected.

### The important detail

The input's displayed value is not really "just whatever was typed." It
is React explicitly setting the input's value to equal `name` on every
render. It happens to match what was typed because `onChange` correctly
updates `name` to match it. Technically, React decides what the input
shows, not the browser's natural typing behavior.

**Proof of this:**

```jsx
<input
  type="text"
  value={name}
  onChange={(e) => setName(e.target.value.toUpperCase())}
/>
```

If a lowercase "a" is typed here, `setName("A")` runs, since the value is
converted to uppercase before being stored. The input will then actually
display "A", not "a," even though a lowercase letter was physically
typed. This proves that React fully controls what is displayed - it is
not simply reflecting whatever was typed. This is the core meaning of a
"controlled component": React's state is the single source of truth for
what is shown, and the browser simply follows it.

---

## Why this control is actually useful

### Formatting input as the user types

Example: automatically formatting a phone number as digits are typed.

```jsx
function PhoneInput() {
  const [phone, setPhone] = useState("");

  function handleChange(e) {
    const digitsOnly = e.target.value.replace(/\D/g, ""); // remove non-digits
    const formatted = digitsOnly.replace(/(\d{3})(\d{3})(\d{4})/, "$1-$2-$3");
    setPhone(formatted);
  }

  return <input value={phone} onChange={handleChange} />;
}
```

Here, whatever is typed gets transformed before being shown back to the
user. The displayed value is never exactly "what was typed" - it is
"what React decided to show after processing it." This only works because
React fully controls the input's value.

### Enforcing a character limit

Example: a text box that should never allow more than 280 characters.

```jsx
function TweetBox() {
  const [text, setText] = useState("");

  function handleChange(e) {
    if (e.target.value.length <= 280) {
      setText(e.target.value);
    }
    // if over 280 characters, state is simply not updated
  }

  return <input value={text} onChange={handleChange} />;
}
```

If a user tries to type past 280 characters, `setText` never runs for
that extra character, so `text` stays at its previous value. Since
`value={text}` controls what is displayed, the extra character never
shows up on screen, even though the browser tried to let it through.

### Keeping multiple inputs in sync

Example: a username field where typing updates a live preview shown
elsewhere on the page, like `yoursite.com/username`. Since the React
state is the single source of truth, that same state value can be used
in multiple places at once, and they will always match, since both are
reading from the same state rather than independently tracking what the
browser did.

**Summary:**
Controlled components let React validate, transform, restrict, or reuse
whatever the user types, because React decides what actually gets shown,
rather than simply watching what the browser naturally displays.