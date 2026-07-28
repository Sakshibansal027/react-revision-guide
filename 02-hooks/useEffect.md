# useEffect

## What is useEffect for?

useEffect lets you run some code after the component shows up on the
screen, or after something changes. This kind of code is called a "side
effect" - it is not about what to show on screen, it is about doing
something extra. Some common examples:

- Getting data from an API
- Starting a timer
- Adding an event listener
- Changing the page title
- Opening a connection to a server

Normally, a component's job is simple: take the current state and props,
and decide what to show. But things like API calls or timers are
different - they do something outside the component. If we wrote this
kind of code directly inside the component, it would run again and again
on every render, with no control. useEffect gives us a way to say: run
this code after the screen updates, and only when we actually want it to
run.

---

## When does useEffect run?

```jsx
useEffect(() => {
  console.log("Component rendered");
});
```

This code is correct. Here is what happens step by step:

- useEffect always runs **after** the screen has already updated, not
  before.
- Since there is no second argument here (no dependency array), this
  code runs after **every single render**, no matter what.

So if the component renders three times, this console.log will run three
times too - once after each render.

---

## The dependency array

The dependency array is the second thing you can pass into useEffect. It
controls how often the code inside runs.

```jsx
useEffect(() => { /* code */ });          // no dependency array
useEffect(() => { /* code */ }, []);       // empty array
useEffect(() => { /* code */ }, [count]);  // array with a value
```

- **No array at all:** the code runs after every render, every time.
  This is usually not what you want, unless you really need it to run on
  every render.

- **Empty array `[]`:** the code runs only one time - right after the
  component shows up on the screen for the first time. It will not run
  again after that, even if the component re-renders many times. This is
  commonly used to load some initial data when the component first
  appears.

- **Array with something inside, like `[count]`:** the code runs once
  after the first render, and then it runs again every time `count`
  changes. If some other value changes but `count` stays the same, the
  code will not run again.

**Simple way to think about it:** the dependency array tells React,
"only run this again if one of these values has changed since last
time." An empty array means nothing to compare, so it just runs once. No
array at all means React has no way to check, so it just runs every
time.

---

## Cleanup functions

Sometimes, the code inside useEffect starts something that needs to be
stopped later - otherwise it keeps running even after we don't need it
anymore. This is what a cleanup function does.

If you return a function from inside useEffect, that becomes the cleanup
function.

```jsx
useEffect(() => {
  // this part is the "setup" code
  const interval = setInterval(() => {
    setTime(new Date());
  }, 1000);

  // this returned function is the "cleanup" code
  return () => {
    clearInterval(interval);
  };
}, []);
```

Cleanup runs in exactly two situations:

### Situation 1: the component disappears from the screen (unmounts)

Example: a Clock component that runs a timer.

1. The user opens the Clock page - useEffect runs, the timer starts, and
   the time updates every second.
2. The user clicks a "Home" button and moves to a different page - the
   Clock component disappears from the screen (this is called
   unmounting).
3. If there was no cleanup function, that `setInterval` would keep
   running in the background, even though the component is gone. It
   would waste memory and could cause errors.
4. Because of this, React automatically runs the cleanup function when
   the component disappears, so `clearInterval` runs and the timer
   properly stops.

### Situation 2: the effect is about to run again (a dependency changed)

If something in the dependency array changes, React first cleans up the
previous effect, and only then runs the new one.

Example with an event listener instead of a timer:

```jsx
useEffect(() => {
  function handleResize() {
    console.log("window resized");
  }

  window.addEventListener("resize", handleResize); // setup: listener added

  return () => {
    window.removeEventListener("resize", handleResize); // cleanup: listener removed
  };
}, []);
```

Without cleanup, if this component mounts and unmounts repeatedly (like a
popup opening and closing again and again), a new listener would get
added every time, but the old one would never be removed. The result:
the same resize event would trigger the function multiple times, which
is incorrect.

**Simple rule to remember:**
- Setup (starting a timer, adding a listener) goes inside useEffect.
- The matching "shut it down" step (stopping the timer, removing the
  listener) goes inside the returned cleanup function.
- React automatically runs the cleanup at the right time - there is
  nothing extra to manually track.

---

## Example: WebSocket connection

A WebSocket is a connection between the browser and a server that stays
open, so data can be sent both ways in real time, without sending a new
request every single time. This is different from a normal request
(like fetch), where the connection closes right after getting a
response.

```jsx
useEffect(() => {
  const socket = new WebSocket("wss://chat-server.com");

  socket.onmessage = (event) => {
    console.log("New message:", event.data);
  };

  return () => {
    socket.close();
  };
}, []);
```

If the component using this connection disappears from the screen, and
we don't close the connection in a cleanup function, it stays open in
the background for no reason, using up resources. If the user opens and
closes this component many times, many connections could pile up at
once, which can slow down or break the app.

---

## Note: this is different from push notifications

It might seem like closing a WebSocket would stop an app like WhatsApp
from receiving messages when it is closed. That is not true, because two
different systems are involved here:

- **A WebSocket connection** is used while the app is open and a
  specific screen (like a chat) is active, to get real-time updates on
  that screen.
- **Push notifications** are a completely separate system. The app
  registers once with the phone's operating system. When a new message
  arrives, the server tells the operating system, and the operating
  system shows the notification directly - whether the app is open or
  not.

Closing a WebSocket connection only affects that one specific real-time
connection for that screen. It does not affect the separate system that
delivers notifications when the app is fully closed.