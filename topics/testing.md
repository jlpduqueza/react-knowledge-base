# Testing React with React Testing Library

## Philosophy

React Testing Library (RTL) encourages testing components the way a user would use them — querying by visible text, labels, and roles rather than internal implementation details (state, instance methods, class names). "The more your tests resemble the way your software is used, the more confidence they can give you."

## Basic Setup

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect } from 'vitest'; // or Jest

test('renders a greeting', () => {
  render(<Greeting name="Ada" />);
  expect(screen.getByText('Hello, Ada')).toBeInTheDocument();
});
```

## Querying Elements

Prefer queries in this priority order (closest to how users/assistive tech find things):

1. `getByRole` — `getByRole('button', { name: /submit/i })`
2. `getByLabelText` — for form fields
3. `getByPlaceholderText`, `getByText`
4. `getByTestId` — last resort, when nothing semantic is available

```jsx
render(<LoginForm />);
const emailInput = screen.getByLabelText(/email/i);
const submitButton = screen.getByRole('button', { name: /log in/i });
```

## Simulating User Interaction

`userEvent` simulates real user interactions (focus, keystrokes, clicks) more accurately than firing raw DOM events directly.

```jsx
test('submits the form with entered values', async () => {
  const user = userEvent.setup();
  render(<LoginForm onSubmit={mockSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'ada@example.com');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(mockSubmit).toHaveBeenCalledWith({ email: 'ada@example.com' });
});
```

## Async Queries

For content that appears after an async action (a fetch, a state update after a delay), use `findBy*` (returns a promise) or wrap assertions in `waitFor`:

```jsx
test('shows results after fetching', async () => {
  render(<SearchResults query="react" />);
  const result = await screen.findByText('React - A JavaScript library');
  expect(result).toBeInTheDocument();
});
```

## Mocking API Calls

```jsx
import { vi } from 'vitest';

vi.mock('./api', () => ({
  fetchUser: vi.fn(() => Promise.resolve({ name: 'Ada' })),
}));

test('displays fetched user', async () => {
  render(<UserProfile userId={1} />);
  expect(await screen.findByText('Ada')).toBeInTheDocument();
});
```

For more realistic API mocking across a whole test suite, **Mock Service Worker (MSW)** intercepts actual network requests instead of mocking modules.

## Testing Custom Hooks

```jsx
import { renderHook, act } from '@testing-library/react';

test('useCounter increments', () => {
  const { result } = renderHook(() => useCounter());

  act(() => { result.current.increment(); });

  expect(result.current.count).toBe(1);
});
```

## What Not to Test

- Internal state values directly — test the rendered output/behavior they produce instead.
- Implementation details like which specific hook was called, or a component's internal method names.
- Third-party library internals (trust that `react-router` or `redux` work; test how your code uses them).

## Key Takeaways

- Query by role/label/text, not by CSS class or test ID, whenever possible.
- Use `userEvent` over `fireEvent` for realistic interaction simulation.
- Use `findBy*`/`waitFor` for anything asynchronous — don't just assume synchronous rendering.
- Tests should survive a refactor that doesn't change user-visible behavior.
