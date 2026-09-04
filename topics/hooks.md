# React Hooks

Hooks let function components use state, side effects, and other React features without writing a class. Rules: only call hooks at the top level (never in loops/conditions/nested functions), and only from React function components or custom hooks.

## useState

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

For updates based on the previous value, use the functional form to avoid stale-state bugs:

```jsx
setCount(prev => prev + 1);
```

## useEffect

Runs side effects (data fetching, subscriptions, manual DOM changes) after render.

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]); // dependency array - re-runs only when `count` changes

useEffect(() => {
  const timer = setInterval(() => tick(), 1000);
  return () => clearInterval(timer); // cleanup, runs before next effect / on unmount
}, []); // empty array - runs once, like componentDidMount
```

| Dependency array | Behavior                                  |
|--------------------|--------------------------------------------|
| Omitted            | Runs after every render                    |
| `[]`               | Runs once, after the first render          |
| `[a, b]`           | Runs after the first render and whenever `a` or `b` changes |

## useContext

Reads a value from context without wrapping children in a consumer component:

```jsx
const ThemeContext = createContext('light');

function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>...</div>;
}

<ThemeContext.Provider value="dark">
  <Toolbar />
</ThemeContext.Provider>
```

## useRef

Holds a mutable value that persists across renders without causing a re-render when changed. Common uses: DOM references, storing a previous value, timers/interval IDs.

```jsx
function TextInput() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);
  return <input ref={inputRef} />;
}
```

## useMemo and useCallback

Both are performance optimizations that skip expensive recomputation between renders when dependencies haven't changed. Don't reach for them by default — only when profiling shows an actual cost.

```jsx
const sortedList = useMemo(() => expensiveSort(list), [list]); // memoizes a value

const handleClick = useCallback(() => {
  doSomething(id);
}, [id]); // memoizes a function reference (useful when passed to memoized children)
```

## useReducer

An alternative to `useState` for more complex state logic, especially when the next state depends on several sub-values or the previous state in structured ways:

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    default: throw new Error('unknown action');
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return <button onClick={() => dispatch({ type: 'increment' })}>{state.count}</button>;
}
```

## Custom Hooks

A custom hook is just a function whose name starts with `use` that calls other hooks — a way to extract and reuse stateful logic across components.

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);

  return width;
}

function ResponsiveComponent() {
  const width = useWindowWidth();
  return <p>Window is {width}px wide</p>;
}
```

## Common Pitfalls

- Missing dependencies in `useEffect`'s array leads to stale closures reading old props/state.
- Calling hooks conditionally breaks React's ability to track hook order between renders.
- Using an array index as a `key` when list order can change causes subtle rendering bugs.
- Overusing `useMemo`/`useCallback` everywhere adds complexity without measurable benefit.
