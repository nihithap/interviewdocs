# React Interview Handbook
### A complete Q&A prep guide covering core React concepts (Classic → React 19)

---

## Table of Contents

1. React Fundamentals & JSX
2. Components, Props & State
3. Core Hooks — `useState` & `useEffect`
4. `useRef`, `useMemo`, `useCallback`
5. `useContext` & the Context API
6. `useReducer`
7. Custom Hooks
8. Component Lifecycle (Class vs Hooks)
9. Virtual DOM & Reconciliation
10. Lists, Keys & Conditional Rendering
11. Event Handling & Synthetic Events
12. Forms — Controlled vs Uncontrolled
13. State Management (Context vs Redux vs Zustand)
14. React Router
15. Performance Optimization
16. Error Boundaries
17. Refs & DOM Manipulation
18. HOCs & Render Props (legacy composition patterns)
19. React 18 — Concurrent Rendering, Suspense, Transitions
20. React 19 — Actions, `use()`, Server Components, the Compiler
21. Testing (Jest & React Testing Library)
22. SSR, Next.js Basics & Hydration

---

# 1. React Fundamentals & JSX

**Q1: What is React, and why is it called a "library" rather than a "framework"?**

A: React is a **declarative, component-based UI library** for building user interfaces — you describe *what* the UI should look like for a given state, and React figures out *how* to update the DOM to match. It's called a library (not a framework) because it deliberately handles only the view layer — routing, state management, HTTP, and build tooling are left to separate libraries you choose and compose yourself (React Router, Redux/Zustand, Axios/fetch, Vite/Webpack), unlike an opinionated full-stack framework that bundles all of this.

**Q2: What is JSX, and how does it actually get converted into JavaScript?**

A: JSX is a syntax extension that lets you write HTML-like markup inside JavaScript. It's **not** understood by browsers directly — a compiler (Babel, or the TypeScript compiler) transforms it into plain `React.createElement()` calls (or, since the React 17+ **new JSX transform**, into calls to an auto-imported `jsx`/`jsxs` function from `react/jsx-runtime`, removing the historical need to `import React` in every file just to use JSX):
```jsx
const element = <h1 className="title">Hello</h1>;
// compiles to (classic transform):
const element = React.createElement('h1', { className: 'title' }, 'Hello');
```
Each `createElement`/`jsx` call returns a plain JavaScript object (a "React element") describing the UI — this is the raw material the Virtual DOM diffing algorithm operates on.

**Q3: Why does JSX use `className` instead of `class`, and `htmlFor` instead of `for`?**

A: Because JSX attributes map to **DOM properties**, not HTML attribute strings, and `class`/`for` are **reserved words** in JavaScript — you can't use them as object property names in the way JSX compiles down to. React uses the camelCase DOM property equivalents instead (`className`, `htmlFor`, `onClick`, `tabIndex`) for nearly all attributes, which is a common early "gotcha" question for candidates coming from plain HTML.

**Q4: What are React Fragments, and why use them instead of a wrapping `<div>`?**

A: `<React.Fragment>` (or the shorthand `<>...</>`) lets a component return multiple sibling elements **without adding an extra DOM node** — useful because a component can only return a single root element, and needlessly wrapping in a `<div>` can break CSS layouts (e.g., flex/grid contexts) or add meaningless markup. The keyed form (`<React.Fragment key={id}>`) is needed specifically when rendering a list of fragments, since the shorthand `<>` syntax can't accept a `key` prop.

---

# 2. Components, Props & State

**Q1: Function components vs Class components — what's the modern recommendation, and why?**

A: **Function components with Hooks** are the modern standard (since React 16.8, 2019) and are what the React team actively recommends and continues to invest new features in (Suspense, Concurrent features, the Compiler are all Hooks/function-component-first). Class components still work (React maintains backward compatibility) but are considered **legacy** — they require more boilerplate (`this` binding, constructor for state init), lack a clean way to share **stateful logic** between components (Hooks solve this cleanly via custom hooks, whereas classes relied on HOCs/render props, which caused "wrapper hell"), and newer APIs (like the `use()` hook) simply aren't available in classes at all.

**Q2: What's the difference between props and state?**

A: **Props** are read-only data passed **into** a component from its parent — a component must never mutate its own props (this is a core React rule, similar to a function not mutating its arguments). **State** is data owned and managed **internally** by a component (via `useState`/`useReducer`), which the component itself can update, triggering a re-render. Props flow **down** the tree; state changes are local unless explicitly lifted up or shared via context/a state library.

**Q3: What is "lifting state up," and when do you need it?**

A: When two or more sibling components need to share/stay in sync on the same piece of state, you move (**lift**) that state to their nearest common ancestor, which then passes the value down as props and passes down callback functions the children can call to request changes. This keeps a single source of truth instead of duplicating state across siblings that could drift out of sync.

**Q4: What's prop drilling, and what are the common ways to avoid it?**

A: Prop drilling is passing a prop through several layers of intermediate components that don't actually use it themselves, purely to get it to a deeply nested descendant — it clutters intermediate components' signatures and makes refactoring fragile. Common fixes: the **Context API** (for state a whole subtree needs, e.g., theme/auth/locale), **component composition** (passing already-rendered JSX as `children`/a prop instead of raw data, so the intermediate component doesn't need to know about the data at all), or a dedicated **state management library** for genuinely global/cross-cutting state.

---

# 3. Core Hooks — `useState` & `useEffect`

**Q1: How does `useState` work, and why must Hooks always be called in the same order on every render ("Rules of Hooks")?**

A: `useState` returns a `[value, setter]` pair and, on subsequent renders, retrieves the current value from React's internal per-component **hooks list**. React identifies which state a given `useState()` call refers to purely by **call order/index** within the component (there's no name/key involved) — so hooks must always be called **unconditionally, at the top level**, in the exact same order every render. Calling a hook inside an `if`, loop, or after an early `return` would shift the index for every hook after it, causing React to read/write the wrong internal slot — this is why ESLint's `eslint-plugin-react-hooks` flags conditional hook calls as an error, not just a style nitpick.

**Q2: Why does calling the `setState` setter not update the variable immediately within the same event handler?**

A: State updates are **asynchronous and batched** (updates queue up and are applied before the next render, not synchronously mid-function) — the `value` you already have in your closure remains the value from the render that closure was created in, since state updates trigger a **new render** with a fresh closure holding the new value. This is a very common gotcha:
```jsx
const [count, setCount] = useState(0);
function handleClick() {
  setCount(count + 1);
  console.log(count); // still logs the OLD value — this line's closure predates the update
}
```

**Q3: What's the difference between `setCount(count + 1)` and `setCount(prev => prev + 1)`, and when does it matter?**

A: The **functional updater form** (`prev => prev + 1`) receives the guaranteed-latest state value at the time the update is actually applied, rather than relying on the `count` captured in the current render's closure. This matters when you call the setter **multiple times in a row** or inside something async (e.g., a `setTimeout`) — `setCount(count + 1); setCount(count + 1);` both read the same stale `count` and only increment by 1 total, while `setCount(p => p+1); setCount(p => p+1);` correctly increments by 2, since each functional update builds on the previous one's result.

**Q4: Explain `useEffect`'s dependency array — what do `[]`, no array, and `[a, b]` each mean?**

A:
- **No dependency array** — the effect runs after **every** render (rarely what you want; easy to cause performance issues or infinite loops if it also sets state unconditionally).
- **`[]`** (empty array) — the effect runs **once**, after the initial mount only (and its cleanup runs once, on unmount) — commonly used for one-time setup like subscriptions or initial data fetches.
- **`[a, b]`** — the effect re-runs whenever `a` or `b` changes (compared via `Object.is`/reference equality) between renders, in addition to running once on mount.

**Q5: What's the cleanup function in `useEffect`, and when does it run?**

A: The function optionally **returned** from an effect callback — React runs it **before** the effect runs again (on a dependency change) and when the **component unmounts**, giving you a place to tear down subscriptions, clear timers, cancel network requests, or remove event listeners set up by that effect, preventing memory leaks and stale-callback bugs:
```jsx
useEffect(() => {
  const id = setInterval(() => setTick(t => t + 1), 1000);
  return () => clearInterval(id); // cleanup
}, []);
```

**Q6: What's the difference between `useEffect` and `useLayoutEffect`?**

A: `useEffect` runs **asynchronously**, after the browser has painted the updated DOM to the screen — good for most side effects (data fetching, subscriptions) since it doesn't block visual updates. `useLayoutEffect` runs **synchronously**, immediately after DOM mutations but **before** the browser paints — necessary when you need to measure/read layout (e.g., an element's size/position via `getBoundingClientRect()`) and then synchronously adjust something before the user sees any visual flicker (e.g., positioning a tooltip based on measured dimensions). Overusing `useLayoutEffect` can hurt performance since it blocks painting.

---

# 4. `useRef`, `useMemo`, `useCallback`

**Q1: What is `useRef` used for, beyond accessing DOM elements?**

A: `useRef` creates a mutable object (`{ current: value }`) that **persists across renders without triggering a re-render** when changed — two distinct uses:
1. **DOM access** — `<input ref={inputRef} />` then `inputRef.current.focus()`.
2. **Mutable instance variable** — storing any value you need to persist between renders but that shouldn't cause a re-render when it changes (e.g., a timer ID, a "previous value" for comparison, a flag tracking whether this is the first render). This is the key distinguishing trait vs `useState`: updating a ref's `.current` does **not** schedule a re-render.

**Q2: What does `useMemo` do, and when should you actually reach for it?**

A: `useMemo(fn, deps)` **memoizes the result of an expensive computation**, only recomputing it when one of the dependencies changes — otherwise it returns the cached value from the previous render:
```jsx
const sortedList = useMemo(() => expensiveSort(items), [items]);
```
Use it for genuinely expensive calculations (heavy filtering/sorting/aggregation of large data), or to preserve **referential equality** of an object/array passed down to a memoized child component (`React.memo`), so that child doesn't re-render unnecessarily just because a new object literal was created on every parent render. Overusing `useMemo` on cheap computations adds overhead (the dependency comparison itself has a cost) without meaningful benefit — it's an optimization, not a default habit.

**Q3: What does `useCallback` do, and how does it differ from `useMemo`?**

A: `useCallback(fn, deps)` memoizes a **function reference** itself (it's essentially `useMemo(() => fn, deps)`), returning the same function instance across renders as long as dependencies haven't changed — preventing a new function object from being created on every render. This matters primarily when passing a callback as a prop to a **memoized child** (`React.memo`) — without `useCallback`, a new function reference every render would defeat `React.memo`'s shallow-comparison optimization, causing the child to re-render anyway even though the "logic" of the function didn't change.

**Q4: Give a concrete example of why skipping `useCallback` can break `React.memo`.**

A:
```jsx
const Child = React.memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  // BAD: new function reference every Parent render, defeats React.memo
  const handleClick = () => console.log('clicked');
  return <Child onClick={handleClick} />;
}
```
Every time `Parent` re-renders (e.g., due to unrelated state changing), `handleClick` is a **brand-new function object**, so `Child`'s shallow prop comparison sees a "changed" `onClick` prop and re-renders despite `React.memo`. Wrapping `handleClick` in `useCallback(() => ..., [])` fixes this by keeping the same reference across renders.

---

# 5. `useContext` & the Context API

**Q1: How does the Context API work, and what problem does it solve?**

A: `createContext()` creates a Context object; a `<Context.Provider value={...}>` wraps a subtree and makes `value` available to any descendant that calls `useContext(Context)`, **regardless of how deeply nested** — bypassing intermediate components entirely (solving prop drilling for cross-cutting concerns like theme, authenticated user, locale, feature flags).
```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}
function Toolbar() { return <ThemedButton />; } // doesn't need to know about theme
function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

**Q2: What's a common performance pitfall with Context, and how do you mitigate it?**

A: **Every** component consuming a context via `useContext` re-renders whenever the Provider's `value` changes — **regardless of `React.memo`** on the consumer (memo only protects against prop changes, not context changes). If you put frequently-changing values in a single large context, all consumers re-render on every change even if they only cared about one slice of that value. Mitigations: **split contexts** by concern/update-frequency (a `UserContext` separate from a `ThemeContext` separate from fast-changing `NotificationCountContext`), memoize the `value` object passed to the Provider (`useMemo`) to avoid creating a new object reference on every parent render, or move to a proper state management library with more granular subscription (Zustand, Redux with selectors) for very frequently updated global state.

**Q3: Is Context a replacement for a state management library like Redux?**

A: Not entirely — Context is a **dependency-injection mechanism** for passing data through the tree, not a state management solution with built-in features like time-travel debugging, middleware, devtools, action logging, or fine-grained/selector-based subscriptions. Context works well for **relatively static or infrequently-changing global data** (theme, auth, i18n). For complex, frequently-updated, or large-scale application state with many independent consumers needing surgical re-render control, a dedicated library (Redux, Zustand, Jotai, Recoil) is usually a better fit.

---

# 6. `useReducer`

**Q1: When would you choose `useReducer` over `useState`?**

A: When state logic is **complex** — multiple sub-values that update together, or the next state depends on the previous state in non-trivial ways, or there are many distinct "types" of updates (common in forms with many fields, or state machines with distinct transitions). `useReducer` centralizes all update logic into a single pure **reducer function**, making state transitions more predictable, easier to test in isolation (a reducer is just a pure function — no rendering needed to test it), and easier to trace/debug (every state change corresponds to a named, dispatched action).

**Q2: Write a basic counter using `useReducer`.**
```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    case 'reset': return { count: 0 };
    default: throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>−</button>
    </>
  );
}
```

**Q3: How does `useReducer` conceptually relate to Redux?**

A: They share the exact same core pattern — a pure `(state, action) => newState` reducer function and `dispatch()`-based updates — because Redux's design directly inspired (and now, `useReducer` is often described as) "Redux's core loop, built into React." The difference is scope: `useReducer` manages state local to a single component (or a tree passed down via context, if you combine it with `useContext` for a mini global-store pattern), while Redux provides a single global store, middleware ecosystem (for async, logging), and devtools designed for app-wide state shared across many disconnected parts of a large application.

---

# 7. Custom Hooks

**Q1: What is a custom hook, and what naming convention must it follow?**

A: A custom hook is just a regular JavaScript function that **calls other hooks internally** and whose name starts with `use` (a convention React's linter relies on to apply the Rules of Hooks correctly to it). Custom hooks are the primary mechanism for **extracting and reusing stateful logic** between components — something classes could never do cleanly (they needed HOCs/render props, which added extra wrapper components to the tree).

**Q2: Write a custom hook `useFetch` for data fetching with loading/error state.**
```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(json => { if (!cancelled) { setData(json); setLoading(false); } })
      .catch(err => { if (!cancelled) { setError(err); setLoading(false); } });
    return () => { cancelled = true; }; // avoid setting state on an unmounted/stale request
  }, [url]);

  return { data, loading, error };
}

// usage:
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`/api/users/${userId}`);
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <div>{data.name}</div>;
}
```
The `cancelled` flag guards against a classic bug: if `url` changes (or the component unmounts) before the fetch resolves, you don't want a stale response overwriting newer state.

**Q3: Do custom hooks share state between the components that use them?**

A: **No** — each call to a custom hook gets its **own independent state**. If `ComponentA` and `ComponentB` both call `useFetch(...)`, they each get their own separate `data`/`loading`/`error` state; the hook is reusable **logic**, not shared **state**. If you need actual shared state across components, you need Context (or a state library) combined with the custom hook, not the custom hook alone.

---

# 8. Component Lifecycle (Class vs Hooks)

**Q1: Map the class lifecycle methods to their Hooks equivalents.**

A:
| Class lifecycle | Hooks equivalent |
|---|---|
| `constructor` | `useState` initializer |
| `componentDidMount` | `useEffect(() => {...}, [])` |
| `componentDidUpdate` | `useEffect(() => {...}, [deps])` |
| `componentWillUnmount` | the cleanup function returned from `useEffect` |
| `shouldComponentUpdate` | `React.memo` (for props) / manual bail-out logic |
| `getDerivedStateFromProps` | calculating during render, or `useMemo`/an effect syncing state from props |
| `componentDidCatch` / `getDerivedStateFromError` | no Hook equivalent — **Error Boundaries must still be class components** |

**Q2: Why can't Error Boundaries be written as function components with Hooks (as of current React)?**

A: There's currently no Hook equivalent for `componentDidCatch`/`getDerivedStateFromError` — catching rendering errors in a subtree still requires a class component implementing those lifecycle methods. This is a commonly-cited exception interviewers like to probe, since it's a place classes remain mandatory even in an otherwise fully-Hooks-based, modern codebase (though libraries like `react-error-boundary` wrap this class internally so you can use it in a function-component-friendly way).

**Q3: How would a single `useEffect` combine the behavior of `componentDidMount`, `componentDidUpdate`, AND `componentWillUnmount`, and why is that actually a conceptual shift, not just a syntax trick?**

A: A `useEffect` with a dependency array conceptually represents **"synchronize this side effect with these values"** rather than "run code at these specific lifecycle moments" — this is a deliberate mental model shift the React team pushed for. Instead of thinking "when did the component mount/update," you think "this effect needs to stay in sync with `[a, b]` — re-run whenever they change, and clean up whatever the previous run set up first." This unification is why a single hook, plus its cleanup function, naturally covers what used to require three separate, easy-to-desynchronize class lifecycle methods.

---

# 9. Virtual DOM & Reconciliation

**Q1: What is the Virtual DOM, and why does it help performance?**

A: The Virtual DOM is a lightweight, in-memory JavaScript representation of the actual DOM tree (a tree of plain objects describing element types, props, and children). Direct DOM manipulation is comparatively **slow** (triggers layout/reflow/repaint); React instead lets you describe the desired UI declaratively, computes a new Virtual DOM tree on state/prop changes, **diffs** it against the previous tree (reconciliation), and applies only the **minimal set of actual DOM mutations** needed to bring the real DOM in sync — batching and minimizing expensive direct DOM operations rather than eliminating the DOM tree traversal cost entirely (a nuance worth mentioning: Virtual DOM isn't "free," it's a tradeoff that's usually a net win for typical UI update patterns).

**Q2: Explain React's diffing algorithm (reconciliation) at a high level, and its key heuristics.**

A: A fully general tree-diff algorithm is O(n³) — too slow for UI updates — so React uses **heuristics** to make it O(n):
1. **Different element types produce different trees** — if a `<div>` becomes a `<span>` at the same position, React doesn't try to diff their children; it tears down the old subtree entirely and builds a new one.
2. **Keys hint at element identity across renders** — for lists, `key` props let React match up which child in the new list corresponds to which child in the old list, even if their order changed, instead of assuming purely positional correspondence.

Without stable keys, React falls back to comparing children by **index**, which can cause incorrect reuse of DOM nodes/state when a list is reordered, filtered, or has items inserted in the middle (see Section 10).

**Q3: What's the difference between "React reconciliation" and "React Fiber"?**

A: **Reconciliation** is the *algorithm/process* of diffing old vs new element trees to determine what changed. **Fiber** (React 16+) is the underlying *reimplementation of React's internal engine* that performs reconciliation — it restructured the internal work into an incremental, interruptible unit-of-work model (a "fiber" per component instance), enabling React to **pause, abort, or prioritize** rendering work rather than blocking the main thread with one uninterruptible synchronous pass. Fiber is the foundational architecture that later made **Concurrent Rendering** (React 18) possible — without Fiber's interruptible work model, features like `startTransition` and Suspense-based rendering couldn't exist.

---

# 10. Lists, Keys & Conditional Rendering

**Q1: Why does React require a `key` prop when rendering lists, and what makes a good key?**

A: Keys give React a **stable identity** for each list item across renders, letting it correctly match old elements to new ones during diffing — without this, React can only guess by position/index, which breaks down when items are added, removed, or reordered. A good key is a **stable, unique identifier intrinsic to the data** (a database ID, a UUID) — not something that changes across renders or re-sorts.

**Q2: Why is using the array index as a `key` considered an anti-pattern, with a concrete example of what breaks?**

A:
```jsx
{items.map((item, index) => <TodoItem key={index} text={item.text} />)}
```
If items are **reordered, inserted at the start, or removed** (not just appended at the end), the index-to-item mapping shifts — React will match the *positions*, not the actual logical items, potentially reusing a DOM node (and any **internal component state**, like an uncontrolled input's typed value, or `useState` inside that list item) for what is now conceptually a *different* item. This causes visually correct-looking but subtly broken UI — e.g., a checkbox's checked state or a text input's contents "sticking" to the wrong row after a reorder/delete. Index keys are only safe when the list is **static and never reordered/filtered/has items inserted in the middle**.

**Q3: What are the common patterns for conditional rendering in JSX?**

A:
```jsx
// && short-circuit — careful: if count is 0, this renders "0", not nothing!
{items.length > 0 && <ItemList items={items} />}

// ternary — for either/or rendering
{isLoggedIn ? <Dashboard /> : <LoginForm />}

// early return for whole-component conditional rendering
function Profile({ user }) {
  if (!user) return null;
  return <div>{user.name}</div>;
}
```
The `&&` pitfall (`{count && <Badge count={count} />}` rendering a stray `0` when `count` is `0`) is a frequently-cited real bug interviewers ask about — fix by explicitly coercing to boolean (`count > 0 && ...`) or using a ternary.

---

# 11. Event Handling & Synthetic Events

**Q1: What is React's `SyntheticEvent`, and why does it exist?**

A: `SyntheticEvent` is React's cross-browser wrapper around the browser's native event object, providing a **consistent API** across different browsers (historically important for smoothing over real inconsistencies in older browsers) and integrating with React's event delegation/batching system. As of React 17+, React attaches a single event listener at the **root DOM container** (rather than `document`, as in earlier versions) and internally dispatches to the right component via event delegation — a deliberate choice that, for example, makes multiple React apps/versions co-existing on the same page safer, since event handling doesn't leak to the shared top-level `document`.

**Q2: How do you prevent default browser behavior in a React event handler, e.g., stopping a form submit from reloading the page?**

A:
```jsx
function Form() {
  function handleSubmit(e) {
    e.preventDefault(); // stops the native browser form-submit page reload
    // ... handle submission logic
  }
  return <form onSubmit={handleSubmit}>...</form>;
}
```

**Q3: How do you pass extra arguments to an event handler beyond the event object itself?**

A:
```jsx
{items.map(item => (
  <button key={item.id} onClick={() => handleDelete(item.id)}>Delete</button>
))}
```
Wrapping in an inline arrow function is the common approach — note this does create a new function reference each render (relevant to the `useCallback`/`React.memo` discussion in Section 4 if this button is a memoized child).

---

# 12. Forms — Controlled vs Uncontrolled

**Q1: What's the difference between controlled and uncontrolled components?**

A: A **controlled** component's form value is fully driven by React state — the input's `value` prop is bound to state, and every keystroke goes through an `onChange` handler that updates that state, making React the single source of truth. An **uncontrolled** component lets the DOM itself hold the current value; React reads it only when needed (typically via a `ref`), similar to traditional HTML forms.
```jsx
// Controlled
function ControlledInput() {
  const [value, setValue] = useState('');
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}

// Uncontrolled
function UncontrolledInput() {
  const inputRef = useRef(null);
  function handleSubmit() { alert(inputRef.current.value); }
  return <input ref={inputRef} defaultValue="" />;
}
```

**Q2: When would you prefer uncontrolled over controlled inputs?**

A: Uncontrolled inputs are simpler for very basic forms (no need to wire up state/handlers per field), integrate more easily with non-React code, and can be more performant for forms with **many** fields where you don't need to react to every keystroke (e.g., live validation on every character) — since controlled inputs re-render on every keystroke. Controlled inputs are preferred whenever you need **real-time validation, conditional formatting/masking, or to dynamically enable/disable other UI** based on the current input value — React needs to actually know the value on every change for that.

**Q3: How does React 19's Actions/`useActionState` change form handling?**

A: React 19 introduces **Actions** — functions (often async) that can be passed directly to a `<form action={...}>` prop, automatically receiving `FormData` and letting React manage pending/error/success state for you, largely replacing the manual `useState` + `onSubmit` + loading-flag boilerplate pattern:
```jsx
function ChangeNameForm({ currentName, updateName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const result = await updateName(formData.get('name'));
      if (result.error) return result.error;
      return null;
    },
    null
  );
  return (
    <form action={submitAction}>
      <input type="text" name="name" defaultValue={currentName} />
      <button disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```
This is covered in more depth in Section 20.

---

# 13. State Management (Context vs Redux vs Zustand)

**Q1: When is local component state (`useState`) enough, vs when do you need Context, vs when do you need a full library like Redux/Zustand?**

A:
- **Local state** — data only one component (or its direct children via props) cares about (a toggle, form input, hover state).
- **Context** — data many components across different parts of the tree need to *read*, that changes relatively infrequently (theme, current user, locale, feature flags).
- **Dedicated state library (Redux, Zustand, Jotai, Recoil)** — complex, frequently-updated, cross-cutting application state with many independent read/write points, where you need fine-grained subscriptions (components re-render only when the **specific slice** they use changes, not on every store update), middleware (logging, persistence, async orchestration), and time-travel debugging/devtools.

**Q2: What are the core building blocks of Redux, and how does data flow through it?**

A: **Store** (single source of truth holding the entire app state tree), **Actions** (plain objects describing what happened, `{ type: 'todos/add', payload }`), **Reducers** (pure functions computing new state from `(state, action)`), and **`dispatch()`** (the only way to trigger a state change — components call `dispatch(action)`, which runs through the reducer(s), producing new state, which subscribed components re-render in response to via `useSelector`). This unidirectional flow (action → reducer → new state → re-render) makes state changes traceable and debuggable, at the cost of more upfront boilerplate than simpler alternatives.

**Q3: How does Redux Toolkit (RTK) reduce classic Redux boilerplate?**

A: RTK (the now-standard, officially recommended way to write Redux) provides `createSlice()` — combining action creators and reducer logic in one place, using **Immer** internally so you can write reducers with seemingly "mutating" syntax (`state.count += 1`) that's actually translated into safe immutable updates behind the scenes — plus `configureStore()` (sensible defaults, devtools + middleware pre-wired) and `createAsyncThunk()` for standardized async action handling, collectively eliminating most of the hand-written switch-statement and manual-immutability boilerplate that made classic Redux notoriously verbose.

**Q4: How does Zustand differ philosophically from Redux?**

A: Zustand is a much more minimal, **hook-based** state library — you define a store as a simple function returning state + actions, and consume it directly via a generated hook (`const count = useStore(state => state.count)`) with **no Provider wrapping required** and no action-type/reducer ceremony. It offers built-in **selector-based subscriptions** (a component only re-renders when the specific slice it selects changes) without the extra `useSelector`/`connect` machinery Redux needs, making it popular for small-to-medium apps wanting Redux-like global state with far less boilerplate — trading away some of Redux's stricter structure/conventions and vast middleware ecosystem in exchange for simplicity.

---

# 14. React Router

**Q1: How do you configure basic routes with React Router (v6+ API)?**

A:
```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/products/:id', element: <ProductDetail /> },
  {
    path: '/admin',
    element: <AdminLayout />,
    children: [
      { path: 'users', element: <UserList /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Q2: How do you read route params and query strings?**

A:
```jsx
import { useParams, useSearchParams } from 'react-router-dom';

function ProductDetail() {
  const { id } = useParams();                 // /products/:id
  const [searchParams] = useSearchParams();    // ?sort=price
  const sort = searchParams.get('sort');
  // ...
}
```

**Q3: How do you implement a protected/private route?**

A: Wrap the protected element in a component that checks auth state and redirects if unauthenticated:
```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? children : <Navigate to="/login" replace />;
}
// usage:
{ path: '/dashboard', element: <ProtectedRoute><Dashboard /></ProtectedRoute> }
```

**Q4: What's the difference between `<Link>` and `<NavLink>`, and why not just use `<a href>`?**

A: A plain `<a href="/page">` triggers a **full page reload**, re-downloading the whole app bundle and losing all client-side state — defeating the purpose of a single-page app. `<Link>` intercepts the click and performs client-side navigation instead (updating the URL via the History API and re-rendering just the matched route component). `<NavLink>` is the same, but automatically applies an "active" styling/class when its path matches the current URL — commonly used for navigation menus that need to highlight the current page.

---

# 15. Performance Optimization

**Q1: What does `React.memo` do, and what's its default comparison behavior?**

A: `React.memo(Component)` memoizes a function component, **skipping re-render** if its props are shallowly equal to the previous render's props (checked via `Object.is` on each prop, one level deep — not a deep comparison). You can pass a custom comparison function as a second argument for non-default equality logic. This is most valuable for components that are (a) relatively expensive to render, and (b) receive the same props often relative to how often their parent re-renders — wrapping *every* component in `memo` indiscriminately adds comparison overhead without meaningful benefit for cheap components.

**Q2: What is code-splitting, and how do `React.lazy` and `Suspense` implement it?**

A: Code-splitting breaks the JS bundle into smaller chunks loaded **on demand**, rather than one large bundle downloaded upfront — reducing initial load time.
```jsx
const AdminPanel = React.lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      {showAdmin && <AdminPanel />}
    </Suspense>
  );
}
```
`React.lazy` wraps a dynamic `import()` (which webpack/Vite automatically splits into a separate chunk), and `Suspense` shows a fallback UI while that chunk is being fetched, swapping to the real component once loaded — commonly applied at the route level (each route's component is lazy-loaded) as the highest-leverage, lowest-effort performance win in a typical app.

**Q3: What causes unnecessary re-renders in React, and how do you diagnose them?**

A: Common causes: a parent re-rendering causes **all** its children to re-render by default (regardless of whether their specific props changed) unless the child is wrapped in `React.memo`; passing new object/array/function literals as props on every render (defeating `memo`'s shallow comparison, as covered in Section 4); overly broad Context values causing all consumers to re-render on any change. Diagnose with the **React DevTools Profiler** (records a render session, shows which components rendered and, critically, **why** — via the "why did this render" flag when enabled), or the DevTools' "Highlight updates when components render" setting for a quick visual signal during development.

**Q4: What is the React Compiler (React 19 era), and how does it change the performance-optimization story?**

A: The **React Compiler** (formerly "React Forget") is a build-time tool that automatically analyzes component code and inserts memoization (`useMemo`/`useCallback`/`React.memo`-equivalent optimizations) **for you**, without needing to hand-write those hooks — it understands component/data dependencies well enough to safely auto-memoize in most cases. This shifts the mental model: instead of manually reasoning about "does this need `useMemo`," most standard React code gets optimized automatically at compile time, and manual memoization becomes an escape hatch for edge cases rather than a routine chore — teams adopting it have reported removing large amounts of hand-written memoization code with performance holding steady or improving.

---

# 16. Error Boundaries

**Q1: What is an Error Boundary, and what errors does it NOT catch?**

A: An Error Boundary is a component (still must be a **class component**, per Section 8) that catches JavaScript errors thrown during **rendering**, in lifecycle methods, and in constructors of its child component tree, and displays a fallback UI instead of crashing the whole app.
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError(error) { return { hasError: true }; }
  componentDidCatch(error, info) { logErrorToService(error, info); }
  render() {
    if (this.state.hasError) return <h1>Something went wrong.</h1>;
    return this.props.children;
  }
}
```
It does **NOT** catch errors in: **event handlers** (use a regular `try/catch` there instead), **asynchronous code** (`setTimeout`, promises — the error happens outside React's render call stack), **server-side rendering**, or errors thrown **within the Error Boundary itself**.

**Q2: Why must event handler errors be caught with `try/catch` instead of relying on an Error Boundary?**

A: Error Boundaries work by wrapping React's rendering process — they catch errors that occur while React is actively rendering the component tree. An event handler runs **outside** of that render cycle (triggered later, asynchronously, by a user interaction), so an error thrown inside `onClick={() => { throw new Error() }}` never passes through the render call stack an Error Boundary is watching — it just becomes an uncaught exception in that handler's execution context, requiring ordinary `try/catch` (or a `.catch()` on a promise) at the call site itself.

---

# 17. Refs & DOM Manipulation

**Q1: What is `forwardRef`, and why is it needed?**

A: By default, a `ref` prop passed to a custom function component is **not** automatically forwarded to an underlying DOM node inside it — refs don't pass through components like normal props. `forwardRef` explicitly opts a component into receiving and forwarding a ref to an inner DOM element:
```jsx
const FancyInput = React.forwardRef((props, ref) => (
  <input ref={ref} className="fancy" {...props} />
));
// usage: <FancyInput ref={myRef} /> now lets myRef.current point to the actual <input> DOM node
```
Note: as of **React 19**, `ref` can be passed as a **regular prop** directly to function components without needing `forwardRef` at all — a notable simplification, since `forwardRef` was widely considered one of the more awkward, boilerplate-y parts of the older API.

**Q2: What is `useImperativeHandle`, and when would you use it?**

A: It customizes what a parent sees when it accesses `ref.current` on a `forwardRef`-wrapped component — instead of exposing the raw DOM node, you can expose a **curated set of imperative methods**:
```jsx
const VideoPlayer = forwardRef((props, ref) => {
  const videoRef = useRef(null);
  useImperativeHandle(ref, () => ({
    play: () => videoRef.current.play(),
    pause: () => videoRef.current.pause(),
    // deliberately doesn't expose the full DOM node — just this controlled API
  }));
  return <video ref={videoRef} {...props} />;
});
```
Use this sparingly — it's an escape hatch for imperative APIs (media playback controls, triggering an animation, focusing/scrolling programmatically) that genuinely can't be expressed declaratively through props, not a general-purpose pattern.

---

# 18. HOCs & Render Props (legacy composition patterns)

**Q1: What is a Higher-Order Component (HOC), and what's a classic example?**

A: A HOC is a **function that takes a component and returns a new, enhanced component** — a pattern for reusing component logic before Hooks existed:
```jsx
function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) return <Spinner />;
    return <WrappedComponent {...props} />;
  };
}
const UserListWithLoading = withLoading(UserList);
```

**Q2: What is the Render Props pattern, and how does it compare to HOCs?**

A: A component that takes a **function as a prop** (or as `children`) and calls it to determine what to render, passing it internal state/logic:
```jsx
<MouseTracker render={({ x, y }) => <p>Mouse at {x}, {y}</p>} />
```
Functionally similar in purpose to HOCs (reusing stateful logic across components) but avoids some HOC pitfalls (like prop name collisions between multiple HOCs wrapping the same component). Both patterns share the downside of **"wrapper hell"** — deeply nested trees of wrapping components in React DevTools, and indirect, harder-to-trace data flow.

**Q3: Why have both patterns largely fallen out of favor since Hooks?**

A: **Custom Hooks** solve the exact same "reuse stateful logic across components" problem far more directly — no extra wrapper components in the tree, no naming collisions between multiple enhancers, no indirection between where a prop is defined and where it's used, and it's just plain function composition rather than a special component-wrapping convention. Interviewers often ask about HOCs/render props specifically to test whether a candidate understands **why** Hooks were introduced — the historical pain points these patterns had are the direct motivation for the Hooks API design.

---

# 19. React 18 — Concurrent Rendering, Suspense, Transitions

**Q1: What is Concurrent Rendering, conceptually, and what does it NOT mean?**

A: Concurrent Rendering means React can **prepare multiple versions of the UI at the same time**, work on rendering **interruptibly** (pausing to handle a higher-priority update, like user input, then resuming or discarding the in-progress render), without blocking the main thread for the whole duration of a large render. It does **not** mean React runs your code in actual parallel threads — it's still single-threaded JavaScript; "concurrent" refers to the ability to interleave and prioritize chunks of rendering work, not literal multi-threading.

**Q2: What does `startTransition` do, and when would you use it?**

A:
```jsx
const [query, setQuery] = useState('');
const [results, setResults] = useState([]);

function handleChange(e) {
  setQuery(e.target.value); // urgent — keep the input responsive
  startTransition(() => {
    setResults(expensiveSearch(e.target.value)); // non-urgent, can be interrupted
  });
}
```
`startTransition` marks a state update as **low priority/non-urgent** — React will keep the UI (e.g., the text input itself) responsive to urgent updates (typing) by potentially interrupting or delaying the transition update (e.g., re-rendering a large results list) if something more urgent comes in. `useTransition` (the hook form) additionally gives you an `isPending` flag to show a loading indicator during the transition.

**Q3: What is `useDeferredValue`, and how does it differ from debouncing?**

A: `useDeferredValue(value)` returns a version of `value` that "lags behind" the real one during urgent updates, letting React render the expensive part of the UI with a slightly stale value while prioritizing more urgent work, then catching up once idle:
```jsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => expensiveSearch(deferredQuery), [deferredQuery]);
```
Unlike a fixed-delay **debounce** (which always waits a set time regardless of device speed/load), `useDeferredValue` is **adaptive** — on a fast device/light load it may barely lag at all, while on a slow device or heavy render it defers more, since it's driven by React's actual scheduling rather than a hardcoded timer.

**Q4: How did Suspense change/expand in React 18?**

A: Suspense (originally introduced for `React.lazy` code-splitting) became a more general mechanism for **"suspending" rendering while waiting for async data**, not just lazy-loaded code — paired with data-fetching libraries/frameworks that support the Suspense pattern (React Query, Relay, Next.js's data fetching), a component can "throw" a promise during render, and the nearest `<Suspense fallback={...}>` boundary shows a loading state until that promise resolves. React 18 also enabled **Suspense on the server** (streaming SSR — sending HTML for ready parts of the page immediately while still-loading parts stream in progressively, rather than waiting for the entire page's data before sending anything).

**Q5: What is Automatic Batching in React 18, and how does it differ from React 17's behavior?**

A: **Batching** means grouping multiple state updates into a single re-render instead of re-rendering once per `setState` call. Before React 18, batching only happened inside React event handlers — state updates inside `setTimeout`, promises, or native event handlers each triggered a **separate** re-render. React 18's **automatic batching** extends this to *all* updates, regardless of where they originate, reducing unnecessary re-renders app-wide by default without any code changes needed.

---

# 20. React 19 — Actions, `use()`, Server Components, the Compiler

**Q1: What are "Actions" in React 19, and what problem do they solve for forms/mutations?**

A: Actions are (often async) functions passed to `<form action={...}>` (or invoked inside a `startTransition`), designed to replace the manual "set loading state → call API → handle success/error → reset loading state" boilerplate every form/mutation handler used to hand-roll. React automatically tracks **pending state**, handles errors, and integrates with concurrent features — reducing a common, repetitive, error-prone pattern into a much smaller amount of code.

**Q2: Explain the three new hooks React 19 introduces around Actions: `useActionState`, `useFormStatus`, `useOptimistic`.**

A:
- **`useActionState(actionFn, initialState)`** — wraps an action, tracking its pending/error/result state automatically, and returns `[state, formAction, isPending]` to bind to a form (replaces the earlier experimental `useFormState`).
- **`useFormStatus()`** — lets a component **nested inside** a `<form>` read that form's pending/submission status **without prop drilling** it down manually — e.g., a reusable `<SubmitButton>` component can disable itself during submission just by calling this hook, with no props needed from the parent form.
- **`useOptimistic(state, updateFn)`** — lets you show an **instantly updated UI** optimistically (e.g., a "like" button appearing pressed immediately) while the real request is still in flight, automatically reverting to the real state if the request ultimately fails — a common real-world use case is showing "Message sent" instantly in a chat UI before server confirmation actually arrives.

**Q3: What is the `use()` API, and how is it different from a normal Hook?**

A: `use()` lets you read the value of a **Promise or a Context** directly during render — and unlike every other Hook, it can be called **conditionally** (inside an `if`, a loop, after an early return), because it's not subject to the same "must be called in the same order every render" constraint that the Rules of Hooks otherwise enforce.
```jsx
function Comments({ commentsPromise }) {
  const comments = use(commentsPromise); // suspends until the promise resolves
  return comments.map(c => <Comment key={c.id} {...c} />);
}
```
When `use()` reads a pending Promise, the component **suspends** (like Suspense-integrated data fetching), re-rendering once resolved — conceptually similar to `await`, but usable directly inside component render logic in a way that integrates with Suspense boundaries.

**Q4: What are React Server Components (RSC), and how do `'use client'`/`'use server'` directives fit in?**

A: Server Components **render entirely on the server** — they can directly access databases/filesystems/secrets, and crucially, ship **zero JavaScript to the client** for that component's own code (only the rendered output is sent), reducing bundle size significantly for content that doesn't need client-side interactivity. Since Server Components are the **default** in RSC-enabled frameworks (like Next.js's App Router), the `'use client'` directive marks a component (and its subtree) as needing to run in the browser (required whenever you use state, effects, or browser-only APIs — i.e., any interactive component). `'use server'` marks a **function** (a Server Action) that a Client Component can call as if it were a local async function, but which actually executes on the server — React handles the network call plumbing for you, eliminating a lot of manual API-route wiring for simple mutations.

**Q5: What is the React Compiler, and how does it relate to (but differ from) the changes covered in Section 15?**

A: Covered in more detail under Performance (Section 15) — briefly re-stated here in the React 19 context: it's a **build-time** tool (not a runtime feature) that automatically inserts memoization into your components by statically analyzing your code, aiming to make manual `useMemo`/`useCallback`/`React.memo` largely unnecessary for the common case. It's being rolled out as an opt-in tool in the React 19 era rather than a hook/API you call directly — it changes *how your code is compiled*, not the code you write.

---

# 21. Testing (Jest & React Testing Library)

**Q1: What's the philosophy behind React Testing Library (RTL), and how does it differ from Enzyme's older approach?**

A: RTL's guiding principle: **"test your software the way users actually use it"** — query the DOM by accessible roles/text/labels (`getByRole`, `getByText`, `getByLabelText`) rather than by internal component implementation details (like Enzyme's older `.state()`/`.instance()`/shallow-rendering-into-internals approach). This makes tests more resilient to refactors (a component's internal state shape or structure can change freely as long as the user-facing behavior stays the same) and, as a side effect, tends to nudge you toward more accessible markup, since RTL's queries are built around accessibility semantics.

**Q2: Write a basic RTL test for a counter component.**
```jsx
import { render, screen, fireEvent } from '@testing-library/react';

test('increments count when button is clicked', () => {
  render(<Counter />);
  const button = screen.getByRole('button', { name: /increment/i });
  fireEvent.click(button);
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**Q3: How do you test a component that fetches data asynchronously?**

A: Use `findBy*` queries (which return a Promise and auto-retry until found, or time out) instead of synchronous `getBy*`, combined with mocking the fetch call:
```jsx
test('displays user name after loading', async () => {
  jest.spyOn(global, 'fetch').mockResolvedValue({
    json: () => Promise.resolve({ name: 'Alice' }),
  });
  render(<UserProfile userId="1" />);
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

**Q4: What's the difference between a unit test, an integration test, and an E2E (end-to-end) test in a typical React app's testing strategy?**

A: **Unit tests** verify a single function/hook/small component in isolation (e.g., testing a custom hook's logic directly, or a pure utility function). **Integration tests** (RTL's sweet spot) render a component tree and interact with it as a user would, verifying multiple pieces work together correctly (a form submitting, a list filtering) without mocking every internal collaborator. **E2E tests** (Cypress, Playwright) run the **actual built app in a real browser** against a real (or realistically mocked) backend, verifying complete user flows across multiple pages — slowest and most brittle to write/maintain, but catch issues unit/integration tests structurally can't (real network behavior, real browser rendering quirks, cross-page navigation flows).

---

# 22. SSR, Next.js Basics & Hydration

**Q1: What is Server-Side Rendering (SSR) in the React context, and why use it?**

A: SSR renders the initial React component tree to **HTML on the server** and sends that fully-formed HTML to the browser, rather than an empty shell that only fills in after the JS bundle downloads and executes. Benefits: faster **perceived load time** (users see content immediately rather than a blank page/spinner), and better **SEO** (crawlers see fully-rendered content without needing to execute JavaScript). Frameworks like **Next.js** and **Remix** provide the SSR infrastructure/routing/data-fetching conventions on top of React itself (React alone doesn't include a full SSR framework — `react-dom/server` provides the low-level rendering APIs these frameworks build on).

**Q2: What is Hydration, and what commonly goes wrong with it?**

A: Hydration is the process where React "attaches" to the already-rendered server HTML on the client — reusing the existing DOM nodes and wiring up event listeners/state, rather than tearing down and re-rendering everything from scratch (which would cause flicker and waste the SSR performance benefit). A common failure mode is a **hydration mismatch** — when the server-rendered HTML doesn't exactly match what the client would render on first render (e.g., code that reads `window`/`localStorage`/`Date.now()` differently between server and client, or non-deterministic random IDs) — React detects the mismatch, logs a warning, and in the mismatched region falls back to client-side re-rendering, which can cause a visible flicker and defeats part of the SSR benefit for that section.

**Q3: What's the difference between Static Site Generation (SSG), Server-Side Rendering (SSR), and Client-Side Rendering (CSR), as commonly discussed in a Next.js context?**

A:
- **CSR** — the classic plain-React SPA approach: ship an (nearly) empty HTML shell, fetch data and render entirely in the browser after JS loads. Slowest initial content, weakest default SEO, but simplest infrastructure (works with any static file host).
- **SSR** — HTML generated **per-request**, on-demand, on the server — always fresh data, but adds server response-time latency per request, and requires a running server (not just static file hosting).
- **SSG** — HTML generated **at build time**, once, then served as a static file (optionally with **Incremental Static Regeneration** to periodically refresh specific pages without a full rebuild) — fastest possible serving (can be fully CDN-cached), but data can be stale between builds/regenerations, making it best suited to content that doesn't change on every request (blogs, marketing pages, product catalogs updated periodically).

**Q4: How do React Server Components (Section 20) relate to traditional SSR — are they the same thing?**

A: They're related but distinct. Traditional SSR renders the **whole component tree** to HTML on the server for the *initial* load, but the same component code still gets shipped to and re-executed on the client during hydration (the server rendering is essentially a head start, not a permanent division of labor). **React Server Components** are a more fundamental split — certain components are designated to run **only ever on the server**, never shipped to or re-executed on the client at all (reducing client bundle size permanently, not just for the first paint), while `'use client'` components handle the interactive parts. RSC and SSR are complementary — a framework like Next.js's App Router uses both together (Server Components for the static/data-heavy parts, SSR to render the initial HTML including any needed Client Components, then hydration for the interactive parts).

---

## Quick-Reference Cheat Sheet

| Topic | One-line takeaway |
|---|---|
| JSX | Compiles to `createElement`/`jsx()` calls; `className` not `class` since these are DOM properties |
| Props vs state | Props flow down, read-only; state is owned/mutable locally, triggers re-render on change |
| `useState` | Hooks rely on call-order — never call conditionally; use functional updater for updates based on previous state |
| `useEffect` | `[]` = mount only; always clean up subscriptions/timers/listeners to avoid leaks |
| `useRef` | Persists across renders WITHOUT triggering re-render; use for DOM access or mutable instance vars |
| `useMemo`/`useCallback` | Memoize value/function reference; mainly useful to preserve referential equality for `React.memo` children |
| Context | Solves prop drilling; ALL consumers re-render on value change, even with `React.memo` — split contexts by update frequency |
| `useReducer` | Same core pattern as Redux, scoped to one component; best for complex/interrelated state transitions |
| Custom hooks | Reusable stateful LOGIC, not shared state — each call gets independent state |
| Virtual DOM | Diffing minimizes real DOM mutations; keys give React stable identity across renders — never use array index for dynamic lists |
| Controlled forms | React state is the source of truth; React 19 Actions reduce manual pending/error boilerplate |
| State management | Local state → Context (infrequent, broad reads) → Redux/Zustand (frequent, granular, complex) |
| Performance | `React.memo` + stable prop references, code-splitting via `lazy`/`Suspense`; React Compiler auto-memoizes in React 19 |
| Error boundaries | Still must be class components; don't catch event handler or async errors |
| React 18 | Concurrent rendering (interruptible, not multi-threaded), `startTransition`, automatic batching everywhere |
| React 19 | Actions + `useActionState`/`useFormStatus`/`useOptimistic`, `use()` (conditional-callable), Server Components, Compiler |
| Testing | RTL queries by accessible role/text, not implementation details; `findBy*` for async assertions |
| SSR/RSC | SSR = HTML head start, still hydrates full JS; RSC = components that NEVER ship JS to the client at all |

---

*Tip: React interviews in 2026 increasingly probe React 19 concepts (Actions, `use()`, Server Components, the Compiler) alongside the "why" behind older patterns like HOCs and class lifecycle methods — be ready to explain not just how a Hook works, but what problem it solves relative to what came before it.*
