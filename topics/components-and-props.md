# Components and Props

## Components Are Just Functions

A component is a JavaScript function that accepts a single `props` object and returns JSX describing what should appear on screen.

```jsx
function UserCard(props) {
  return <div className="card">{props.name} - {props.role}</div>;
}

// Usage
<UserCard name="Ada Lovelace" role="Engineer" />
```

## Destructuring Props

```jsx
function UserCard({ name, role, isOnline = false }) {
  return (
    <div className="card">
      {name} - {role} {isOnline ? "(online)" : ""}
    </div>
  );
}
```

`isOnline = false` is a default value used when the prop isn't passed.

## Props Are Read-Only

A component must never modify its own props — data flows one way, parent to child. If a child needs to change something, the parent passes down a callback function as a prop.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return <Child count={count} onIncrement={() => setCount(c => c + 1)} />;
}

function Child({ count, onIncrement }) {
  return <button onClick={onIncrement}>Count: {count}</button>;
}
```

## The children Prop

Anything nested inside a component's JSX tags is passed as `props.children`:

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card>
  <h2>Title</h2>
  <p>Some content</p>
</Card>
```

## Composition Over Inheritance

React favors composing components together (passing components as props/children) instead of class inheritance hierarchies.

```jsx
function Dialog({ title, children }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      {children}
    </div>
  );
}

function ConfirmDialog() {
  return (
    <Dialog title="Are you sure?">
      <button>Yes</button>
      <button>Cancel</button>
    </Dialog>
  );
}
```

## Conditional Rendering Patterns

```jsx
function StatusBadge({ status }) {
  if (status === 'loading') return <Spinner />;
  if (status === 'error') return <ErrorMessage />;
  return <Content />;
}
```

## Lifting State Up

When two sibling components need to share state, move that state to their closest common parent and pass it down via props, rather than trying to sync state between siblings directly.

## Component Naming and File Organization

- Component names are PascalCase (`UserCard`, not `userCard`) — React uses the capitalization to distinguish a custom component (`<UserCard />`) from a built-in DOM tag (`<div />`).
- One component per file is a common convention, named to match (`UserCard.jsx`).

## PropTypes / TypeScript

Plain JS React doesn't check prop types at compile time. Two common ways to add safety:

```jsx
// prop-types (runtime checking)
UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  role: PropTypes.string,
};

// TypeScript (compile-time checking, preferred in modern codebases)
type UserCardProps = { name: string; role?: string };
function UserCard({ name, role }: UserCardProps) { ... }
```
