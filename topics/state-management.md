# State Management in React

## Local Component State

The default choice: `useState`/`useReducer` inside the component that owns the data. Use this until you have a concrete reason to reach for something bigger.

```jsx
function SearchBox() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

## Lifting State Up

When multiple components need the same state, move it to their closest common ancestor and pass it (and setters) down as props.

```jsx
function App() {
  const [cart, setCart] = useState([]);
  return (
    <>
      <ProductList onAdd={item => setCart([...cart, item])} />
      <CartSummary items={cart} />
    </>
  );
}
```

This works well until props have to pass through many layers that don't otherwise need the data ("prop drilling") — that's the signal to consider Context or a state library.

## Context API

Shares state across the tree without manually threading props through every level. Best for state that's genuinely global-ish (theme, current user, locale) — not a replacement for all shared state, since every consumer re-renders when the context value changes.

```jsx
const CartContext = createContext(null);

function CartProvider({ children }) {
  const [cart, setCart] = useState([]);
  return (
    <CartContext.Provider value={{ cart, setCart }}>
      {children}
    </CartContext.Provider>
  );
}

function CartSummary() {
  const { cart } = useContext(CartContext);
  return <p>{cart.length} items</p>;
}
```

## When Context Isn't Enough

Context re-renders every consumer on any value change and has no built-in way to select just a slice of state, or to handle complex async update logic cleanly. Dedicated state libraries fill that gap for larger apps.

## Common State Libraries (Overview)

| Library    | Style                                    | Good for |
|-------------|-------------------------------------------|----------|
| **Redux / Redux Toolkit** | Single global store, actions/reducers, strict unidirectional flow | Large apps, complex shared state, strong dev tooling (time-travel debugging) |
| **Zustand** | Minimal hook-based store, little boilerplate | Small-to-medium apps wanting simplicity over Redux's ceremony |
| **Jotai / Recoil** | Atomic state - small independent pieces of state composed together | Fine-grained reactivity, avoiding unnecessary re-renders |
| **React Query / TanStack Query** | Server state - caching, refetching, invalidation | Data fetched from an API (not really "client state" at all) |

## Server State vs. Client State

A key distinction:
- **Client state** — UI state that lives only in the browser (a form's current values, whether a modal is open).
- **Server state** — data that actually lives on a server and is fetched/cached/synced (a list of orders, a user profile). Tools like React Query treat this differently from `useState`, handling caching, background refetching, and staleness for you instead of hand-rolling it with `useEffect`.

## A Simple Redux Toolkit Slice (for reference)

```js
const cartSlice = createSlice({
  name: 'cart',
  initialState: [],
  reducers: {
    addItem: (state, action) => { state.push(action.payload); },
    removeItem: (state, action) =>
      state.filter(item => item.id !== action.payload),
  },
});

export const { addItem, removeItem } = cartSlice.actions;
```

## Rule of Thumb

Start with local state. Lift it when siblings need it. Reach for Context when it's truly cross-cutting and infrequently updated. Reach for a state library only when Context's re-render cost or lack of structure becomes a real problem — and use a server-state tool (React Query, SWR) for anything that comes from an API, rather than modeling it as ordinary client state.
