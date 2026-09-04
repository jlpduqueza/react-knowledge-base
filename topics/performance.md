# Performance Optimization in React

Most React apps don't need manual optimization — measure first (React DevTools Profiler) before reaching for any of this. Premature memoization adds complexity without benefit.

## Why Re-renders Happen

A component re-renders when its state changes, its parent re-renders (by default, children re-render too), or its context value changes. Unnecessary re-renders of expensive subtrees are the usual performance culprit, not React itself being slow.

## React.memo

Skips re-rendering a component if its props haven't changed (shallow comparison):

```jsx
const ExpensiveRow = React.memo(function ExpensiveRow({ item }) {
  return <tr>{item.name}</tr>;
});
```

Only helps if the component actually re-renders often with the same props, and the render itself is non-trivial — wrapping everything in `memo` by default adds comparison overhead for little gain.

## useMemo

Caches an expensive computed value between renders:

```jsx
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a.price - b.price);
}, [items]);
```

## useCallback

Caches a function reference so it doesn't change identity on every render — mainly useful so a memoized child (`React.memo`) doesn't re-render just because it received a "new" function prop:

```jsx
const handleClick = useCallback(() => {
  onSelect(item.id);
}, [item.id, onSelect]);
```

## Avoiding the Real Cost: Inline Object/Array Literals

```jsx
// New object every render - breaks React.memo on the child
<Chart options={{ color: 'blue' }} />

// Stable reference - defined outside render or memoized
const chartOptions = { color: 'blue' };
<Chart options={chartOptions} />
```

## Virtualization for Long Lists

Rendering thousands of DOM nodes at once is slow regardless of memoization. Libraries like `react-window` or `react-virtualized` render only the visible rows:

```jsx
import { FixedSizeList } from 'react-window';

<FixedSizeList height={400} itemCount={10000} itemSize={35} width={300}>
  {({ index, style }) => <div style={style}>Row {index}</div>}
</FixedSizeList>
```

## Code Splitting and Lazy Loading

Split the bundle so users only download the code for the page/feature they're using:

```jsx
const SettingsPage = React.lazy(() => import('./SettingsPage'));

<Suspense fallback={<Spinner />}>
  <SettingsPage />
</Suspense>
```

## Avoiding Unnecessary Context Re-renders

Every consumer of a context re-renders when its value changes, even if they only use part of it. Split large contexts into smaller, more focused ones, or memoize the provider's value:

```jsx
const value = useMemo(() => ({ user, theme }), [user, theme]);
<AppContext.Provider value={value}>{children}</AppContext.Provider>
```

## Key Takeaways

- Profile before optimizing — React DevTools Profiler shows which components re-render and why.
- `React.memo`/`useMemo`/`useCallback` only help when paired correctly (a memoized child needs stable props to actually skip re-rendering).
- For large lists, virtualization beats memoization.
- Code-split by route/feature to keep the initial bundle small.
