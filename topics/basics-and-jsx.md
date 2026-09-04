# React Basics and JSX

## What React Is

A JavaScript library for building user interfaces out of composable, reusable **components**. React manages a virtual representation of the UI and efficiently updates the real DOM when data changes.

## JSX

JSX is a syntax extension that lets you write HTML-like markup inside JavaScript. It compiles to `React.createElement()` calls.

```jsx
const element = <h1>Hello, world!</h1>;

// Compiles roughly to:
const element = React.createElement('h1', null, 'Hello, world!');
```

### Embedding Expressions

```jsx
const name = "Ada";
const element = <h1>Hello, {name}!</h1>;
const sum = <p>2 + 2 = {2 + 2}</p>;
```

### JSX Rules

- Must return a single root element (or use a Fragment `<>...</>`).
- Use `className` instead of `class`, `htmlFor` instead of `for`.
- Every tag must be closed (`<img />`, not `<img>`).
- JavaScript expressions go in `{}`; statements (if/for) do not work directly — use ternaries, `&&`, or `.map()`.

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <p>Welcome back!</p> : <p>Please sign in.</p>}
      {isLoggedIn && <button>Log out</button>}
    </div>
  );
}
```

## Rendering

```jsx
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

## Components: Function Components

Modern React is written almost entirely with function components (class components still work but are legacy for new code):

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}</h1>;
}

// Equivalent arrow function form
const Welcome = ({ name }) => <h1>Hello, {name}</h1>;
```

## Lists and Keys

```jsx
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.label}</li>
      ))}
    </ul>
  );
}
```

`key` must be a stable, unique identifier (not the array index, when the list can reorder or change) — it's how React matches elements across re-renders to update the DOM efficiently instead of re-creating everything.

## Fragments

Group multiple elements without adding an extra DOM node:

```jsx
function Pair() {
  return (
    <>
      <td>First</td>
      <td>Second</td>
    </>
  );
}
```

## The Virtual DOM (Conceptually)

React keeps a lightweight in-memory representation of the UI tree. On a state change, it re-renders the affected components, diffs the new tree against the previous one, and applies only the minimal set of real DOM updates needed — avoiding costly full-page re-renders.
