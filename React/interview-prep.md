# React — Interview Prep

## Table of Contents
- [1. Core Concepts](#1-core-concepts)
- [2. Components](#2-components)
- [3. Hooks](#3-hooks)
- [4. Re-renders & Performance](#4-re-renders--performance)
- [5. State Management](#5-state-management)
- [6. Lifecycle & Patterns](#6-lifecycle--patterns)
- [7. Thought-Provoking Questions](#7-thought-provoking-questions)

---

## 1. Core Concepts

**Q: What is React and what problem does it solve?**

React is a JavaScript library for building user interfaces through a **component-based model**. The core insight is that UI is a function of state — when state changes, React figures out the minimal DOM updates needed. Before React, developers manually synced data with the DOM (error-prone and slow). React replaces that with a **declarative** model: you describe *what* the UI should look like for a given state, and React handles *how* to update the DOM.

Two key innovations:
- **Virtual DOM** — a lightweight in-memory representation of the real DOM. React diffs the virtual DOM before and after a state change, then applies only the necessary real DOM updates (reconciliation).
- **Component model** — UI broken into composable, reusable pieces, each with its own state and logic.

---

**Q: What is JSX and what does it actually compile to?**

JSX is a syntax extension that looks like HTML inside JavaScript. Babel transforms it into `React.createElement()` calls.

```jsx
// JSX
const el = <h1 className="title">Hello, {name}</h1>;

// What it compiles to
const el = React.createElement("h1", { className: "title" }, "Hello, ", name);
```

JSX is not HTML — key differences:
- `class` → `className`, `for` → `htmlFor`
- All tags must be closed (`<br />`)
- Event handlers are camelCase: `onClick`, `onChange`
- JavaScript expressions go inside `{}`
- A component must return a single root element (or a Fragment `<>...</>`)

---

**Q: What is the Virtual DOM and how does reconciliation work?**

The **Virtual DOM** is a plain JavaScript object tree that mirrors the real DOM. When state or props change:

1. React renders a new virtual DOM tree.
2. **Diffing** — React compares (diffs) the new tree with the previous one.
3. **Reconciliation** — React computes the minimal set of real DOM changes needed.
4. React commits those changes to the real DOM in one batch.

Direct DOM manipulation is expensive. Updating a JS object is cheap. The diff/patch pattern means most renders result in minimal real DOM operations.

React uses **keys** on list items to track element identity across renders. Without a stable key, React can't tell if a list item was moved or replaced — causing incorrect re-renders or lost state.

---

## 2. Components

**Q: Functional vs Class components — what's the difference and which should you use?**

**Class components** (legacy):
- Extend `React.Component`
- State in `this.state`, update with `this.setState()`
- Lifecycle methods: `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`

**Functional components** (modern):
- Plain JavaScript functions that return JSX
- State and lifecycle via **hooks** (`useState`, `useEffect`, etc.)
- Less boilerplate, easier to test, easier to reason about

Since React 16.8 (hooks), **always use functional components**. Class components are maintained but not the recommended path for new code.

```jsx
// Modern functional component
function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

**Q: What are props? Can you modify them?**

**Props** (properties) are the inputs passed from a parent component to a child. They are **read-only** — a component must never modify its own props. This is a core rule: React uses one-way data flow (parent → child).

```jsx
function Greeting({ name, age }) {          // destructured from props
    return <p>Hello {name}, you are {age}</p>;
}

<Greeting name="Alice" age={30} />
```

If a child needs to communicate back to a parent, the parent passes a **callback function** as a prop, and the child calls it.

`children` is a special prop — whatever JSX is nested between the opening and closing tags of a component.

---

**Q: What is the difference between controlled and uncontrolled components?**

- **Controlled** — React state is the single source of truth for the input's value. Every keystroke updates state, which re-renders the input. You always know the current value.

```jsx
function Form() {
    const [value, setValue] = useState("");
    return <input value={value} onChange={e => setValue(e.target.value)} />;
}
```

- **Uncontrolled** — the DOM manages the input's own state. You read the value imperatively via a `ref` when needed (e.g., on submit).

```jsx
function Form() {
    const inputRef = useRef();
    const handleSubmit = () => console.log(inputRef.current.value);
    return <input ref={inputRef} />;
}
```

Prefer **controlled** for most cases — React stays in control of the data. Use uncontrolled for integrating with non-React code or file inputs.

---

## 3. Hooks

**Q: What is `useState` and how does it work?**

`useState` adds local state to a functional component. Returns `[currentValue, setterFn]`.

```jsx
const [count, setCount] = useState(0);       // initial value = 0
const [user, setUser] = useState(null);
const [form, setForm] = useState({ name: "", email: "" });
```

**Key rules:**
- Calling the setter triggers a **re-render** of the component.
- State updates may be **batched** (React 18+: always batched). Don't assume state changes are immediate.
- When new state depends on current state, use the **functional updater** form:

```jsx
setCount(prev => prev + 1);  // safe — always uses latest value
// vs
setCount(count + 1);  // risky in async contexts — may use stale value
```

- State is replaced (not merged) — when updating objects, spread the old state:

```jsx
setForm(prev => ({ ...prev, name: "Alice" }));
```

---

**Q: What is `useEffect` and what is its dependency array?**

`useEffect` lets you run **side effects** after render — data fetching, subscriptions, DOM manipulation, timers.

```jsx
useEffect(() => {
    // runs after render
    document.title = `Count: ${count}`;

    return () => {
        // cleanup: runs before next effect or on unmount
        document.title = "App";
    };
}, [count]);  // only re-run when count changes
```

**Dependency array:**
- `[]` — run once after mount (like `componentDidMount`).
- `[a, b]` — run after mount and whenever `a` or `b` changes.
- No array — run after every render (almost never what you want).

**Common mistake:** missing a dependency in the array. The effect closes over stale values and behaves unexpectedly. ESLint's `exhaustive-deps` rule catches this.

---

**Q: What is `useRef` and when would you use it over `useState`?**

`useRef` returns a mutable object `{ current: value }` that **persists across renders but does NOT trigger a re-render when changed**.

Use it when:
- You need to access a DOM element directly (`ref={myRef}`)
- You need to store a mutable value that shouldn't cause a re-render (e.g., a timer ID, a previous value, an abort controller)

```jsx
function Stopwatch() {
    const [elapsed, setElapsed] = useState(0);
    const intervalRef = useRef(null);   // stores timer ID — no re-render needed

    const start = () => {
        intervalRef.current = setInterval(() => setElapsed(e => e + 1), 1000);
    };
    const stop = () => clearInterval(intervalRef.current);

    return <>
        <p>{elapsed}s</p>
        <button onClick={start}>Start</button>
        <button onClick={stop}>Stop</button>
    </>;
}
```

---

**Q: What is `useMemo` and `useCallback`? When should you actually use them?**

Both are **performance optimizations** that memoize values to avoid recomputation/re-creation on every render.

- **`useMemo`** — memoizes the result of a computation.
- **`useCallback`** — memoizes a function reference.

```jsx
const sortedList = useMemo(
    () => [...items].sort((a, b) => a.price - b.price),
    [items]   // only re-sort when items changes
);

const handleClick = useCallback(
    (id) => dispatch({ type: "DELETE", id }),
    [dispatch]  // stable reference — won't cause child re-renders
);
```

**When to use:** Only when you have a measurable performance problem.
- Don't add them preemptively — they add overhead (comparison cost) and code complexity.
- `useCallback` is most valuable when passing callbacks to memoized child components (`React.memo`), preventing unnecessary re-renders.
- `useMemo` is most valuable for genuinely expensive computations or for preserving referential identity of objects/arrays used in dependency arrays.

---

**Q: What are custom hooks?**

A custom hook is a function that starts with `use` and calls other hooks. It extracts stateful logic into a reusable function.

```jsx
function useFetch(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        let cancelled = false;
        fetch(url)
            .then(res => res.json())
            .then(d => { if (!cancelled) { setData(d); setLoading(false); } })
            .catch(e => { if (!cancelled) { setError(e); setLoading(false); } });
        return () => { cancelled = true; };  // cleanup: ignore response if unmounted
    }, [url]);

    return { data, loading, error };
}

// Usage
function UserProfile({ id }) {
    const { data: user, loading, error } = useFetch(`/api/users/${id}`);
    if (loading) return <Spinner />;
    if (error) return <Error />;
    return <div>{user.name}</div>;
}
```

Custom hooks make logic shareable across components without prop drilling or HOCs.

---

## 4. Re-renders & Performance

**Q: When does a React component re-render?**

A component re-renders when:
1. Its **state** changes (via `setState` or a hook setter).
2. Its **props** change (parent re-renders and passes new values).
3. Its **context** value changes (if it consumes that context).
4. The **parent** re-renders — by default, all children re-render too, even if their props didn't change.

Point 4 is the main source of unnecessary re-renders. Fix with:
- **`React.memo`** — wraps a functional component; skips re-render if props are shallowly equal.
- **`useCallback`/`useMemo`** — stabilize the references passed as props.

```jsx
const Child = React.memo(function Child({ onClick }) {
    console.log("Child rendered");
    return <button onClick={onClick}>Click</button>;
});
```

---

**Q: What is the problem with using array index as a key?**

Keys help React identify which items in a list changed, were added, or removed. Using index as key breaks this when the list is reordered, filtered, or items are inserted/removed from the middle — React can't tell the difference between "item moved" and "item replaced," leading to:
- Wrong component state attached to the wrong item
- Unnecessary full re-renders instead of moves

Always use a **stable, unique identifier** from your data as the key (e.g., `item.id`). Use index only for static lists that never change order.

---

## 5. State Management

**Q: What is prop drilling and how do you solve it?**

**Prop drilling** is passing props down through multiple layers of components just to get data to a deeply nested child that needs it — even though intermediate components don't use it themselves.

**Solutions:**
1. **React Context** — make data available to any component in the tree without explicit prop passing.
2. **State management libraries** — Redux, Zustand, Jotai for complex global state.
3. **Component composition** — restructure to move the consuming component closer to the data.

---

**Q: What is Context and when should you use it?**

Context provides a way to share a value (theme, auth user, locale) across the component tree without prop drilling.

```jsx
const ThemeContext = createContext("light");

function App() {
    const [theme, setTheme] = useState("dark");
    return (
        <ThemeContext.Provider value={theme}>
            <Toolbar />
        </ThemeContext.Provider>
    );
}

function Button() {
    const theme = useContext(ThemeContext);   // no props needed
    return <button className={theme}>Click</button>;
}
```

**When NOT to use Context for state:** Context re-renders every consumer when its value changes. For frequently changing state (e.g., a counter), this can cause performance problems. Context is best for **low-frequency, broadly-shared** values (auth state, theme, locale). For complex or frequently updated state, reach for Zustand or Redux.

---

## 6. Lifecycle & Patterns

**Q: How do you replicate componentDidMount, componentDidUpdate, and componentWillUnmount with hooks?**

```jsx
useEffect(() => {
    // componentDidMount equivalent
    console.log("mounted");
    fetchData();

    return () => {
        // componentWillUnmount equivalent
        console.log("unmounted — cleanup here");
        subscription.unsubscribe();
    };
}, []);  // empty array = run once on mount


useEffect(() => {
    // componentDidUpdate equivalent — runs when userId changes
    fetchUser(userId);
}, [userId]);
```

For `componentDidUpdate` with a "previous value" comparison:
```jsx
const prevCount = useRef(count);
useEffect(() => {
    if (prevCount.current !== count) {
        console.log(`changed from ${prevCount.current} to ${count}`);
        prevCount.current = count;
    }
});
```

---

**Q: What is `useReducer` and when would you use it instead of `useState`?**

`useReducer` manages state via a reducer function — the same pattern as Redux. Better than `useState` when:
- State has complex shape with multiple sub-values
- Next state depends on previous state in a non-trivial way
- Multiple state transitions share the same logic

```jsx
function reducer(state, action) {
    switch (action.type) {
        case "INCREMENT": return { ...state, count: state.count + 1 };
        case "RESET":     return { count: 0 };
        default:          return state;
    }
}

function Counter() {
    const [state, dispatch] = useReducer(reducer, { count: 0 });
    return <>
        <p>{state.count}</p>
        <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
        <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
    </>;
}
```

---

## 7. Thought-Provoking Questions

**Q: If a parent re-renders, does that always mean its children re-render too?**

By default, yes — React re-renders all children of a re-rendering parent. But React.memo can prevent this for functional components when props haven't changed. However, even with React.memo, if you pass a new function or object reference on every render (a common mistake), the memo check fails and the child re-renders anyway — which is why `useCallback` and `useMemo` exist.

---

**Q: Why shouldn't you mutate state directly?**

React compares old and new state by reference to decide whether to re-render. If you mutate the existing state object/array in place, the reference doesn't change — React sees the same reference and skips the re-render, even though the data changed. You'd end up with a stale UI.

```jsx
// Wrong — mutates in place, React won't see the change
state.items.push(newItem);
setState(state);

// Right — new reference, React detects the change
setState(prev => ({ ...prev, items: [...prev.items, newItem] }));
```

---

**Q: What is the Strict Mode and what does it actually do?**

`<React.StrictMode>` wraps your app and in development mode:
- **Double-invokes** render functions, state initializers, and some hooks to help detect side effects (mutations during render, missing cleanup).
- **Warns** about deprecated lifecycle methods and legacy APIs.

In production, Strict Mode has **zero effect**. The double invocation in dev can confuse developers (e.g., `useEffect` appears to run twice) — this is intentional to surface impure code.

```jsx
root.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>
);
```
