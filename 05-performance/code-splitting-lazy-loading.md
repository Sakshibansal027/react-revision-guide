# Code Splitting and Lazy Loading

## The problem: one giant JavaScript bundle

Normally, when a React app is built, all components get bundled into one
large JavaScript file, which the browser downloads before it can show
anything. If an app has 50 pages, but a user only ever visits the
homepage, downloading the code for all 50 pages upfront is wasted
bandwidth and time. It makes the initial page load slower, since the
browser has to download, parse, and process a much bigger file before
showing anything, even though most of that code will not be used during
this visit at all.

---

## Normal import vs React.lazy with dynamic import

### Normal import

```jsx
import HeavyDashboard from "./HeavyDashboard";

function App() {
  return <HeavyDashboard />;
}
```

This tells the build tool (like Vite or webpack) to bundle
`HeavyDashboard`'s code together with everything else, into the main
JavaScript file, upfront. This code gets downloaded immediately, as part
of the initial page load, whether or not the user ever actually visits
that part of the app.

### React.lazy with dynamic import

```jsx
const HeavyDashboard = React.lazy(() => import("./HeavyDashboard"));

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <HeavyDashboard />
    </Suspense>
  );
}
```

This tells the build tool something different: do not bundle this into
the main file. Instead, keep `HeavyDashboard`'s code in its own separate
file (called a chunk), and only download that file when it is actually
needed - the first time this component actually gets rendered.

### The timing difference

- **Normal import:** `HeavyDashboard`'s code is downloaded immediately,
  as part of the initial page load, even if the user never visits the
  dashboard at all during their session.
- **Lazy version:** `HeavyDashboard`'s code is not downloaded at first.
  The main app loads faster, with a smaller initial bundle. Only when
  `<HeavyDashboard />` actually needs to render (for example, the user
  navigates to `/dashboard`), React triggers a separate network request
  to fetch just that chunk of code, on demand.

This overall approach - splitting one giant bundle into multiple smaller
files, each downloaded only when needed - is called **code splitting**.

---

## What Suspense and fallback are for

```jsx
<Suspense fallback={<p>Loading...</p>}>
  <HeavyDashboard />
</Suspense>
```

Since a lazy component's code needs to be fetched over the network the
first time it renders, there is a brief moment where the code has not
arrived yet. Without a fallback, React would have nothing to render at
that spot, since the component does not exist in memory yet - its code
has not even arrived. This is not just confusing for the user; it can
cause an error, or leave a completely blank space on the screen, making
the app look broken or frozen.

`Suspense` lets a fallback be shown while the code is still downloading -
in this case, a simple "Loading..." message. Once the code finishes
downloading, React swaps the fallback out and renders the actual
`HeavyDashboard` component.

---

## A simple analogy

Think of a large streaming app like Netflix. It does not download every
single show's video file the moment the app opens - that would be
absurd. It only downloads the video for the specific show that gets
clicked on, at the moment it is clicked. Code splitting works the same
way for an app's JavaScript - only load what is actually being used,
right when it is needed.

---

## When this is genuinely useful

- **Splitting code by route or page** - the most common use case.
  Someone visiting a login page does not need the code for an admin
  dashboard downloaded upfront.
- **Splitting out genuinely heavy components** that are not always
  needed - like a complex chart library, a rich text editor, or a modal
  that is rarely opened.

---

## Why smaller projects often don't use this

In smaller or medium-sized projects, the total JavaScript bundle size is
often small enough that it downloads in milliseconds on a normal
connection, so there is no noticeable delay for the user. Code splitting
becomes genuinely useful once an app has many pages, some very heavy
components, or a large number of production users where even small
improvements in load time matter for the business.

This is similar to a small shop not needing an inventory management
system, while a company with millions of products genuinely needs one -
the concept is valid either way, but the need for it depends on scale.

For interviews, understanding what code splitting is and when it should
be used matters more than whether it was used in every personal project.