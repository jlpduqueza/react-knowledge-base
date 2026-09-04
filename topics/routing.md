# Routing with React Router

React itself has no built-in router — client-side routing (mapping URLs to components without a full page reload) is handled by a library, most commonly **React Router**.

## Basic Setup

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<UserProfile />} />
        <Route path="*" element={<NotFound />} /> {/* catch-all */}
      </Routes>
    </BrowserRouter>
  );
}
```

## Route Parameters

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  return <p>Viewing user {id}</p>;
}
```

## Navigation

```jsx
import { Link, NavLink, useNavigate } from 'react-router-dom';

// Declarative navigation
<Link to="/about">About</Link>
<NavLink to="/about" className={({ isActive }) => isActive ? 'active' : ''}>About</NavLink>

// Programmatic navigation
function LoginForm() {
  const navigate = useNavigate();
  const handleSubmit = async () => {
    await login();
    navigate('/dashboard');
  };
}
```

`Link` renders an `<a>` tag but intercepts the click to avoid a full page reload; always prefer it over a plain `<a href>` for in-app navigation.

## Nested Routes and Layouts

```jsx
<Routes>
  <Route path="/dashboard" element={<DashboardLayout />}>
    <Route index element={<DashboardHome />} />
    <Route path="settings" element={<Settings />} />
  </Route>
</Routes>
```

```jsx
function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <Outlet /> {/* renders the matched child route here */}
    </div>
  );
}
```

## Query Parameters

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get('category');

  return (
    <button onClick={() => setSearchParams({ category: 'shoes' })}>
      Filter shoes
    </button>
  );
}
```

## Protected Routes

A common pattern: wrap the element in a component that checks auth and redirects if needed.

```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  if (!isAuthenticated) return <Navigate to="/login" replace />;
  return children;
}

<Route path="/dashboard" element={
  <ProtectedRoute><Dashboard /></ProtectedRoute>
} />
```

## Data Loading (React Router 6.4+)

Newer versions support loading data before a route renders, via `loader` functions, reducing waterfall fetches inside `useEffect`:

```jsx
const router = createBrowserRouter([
  {
    path: '/users/:id',
    element: <UserProfile />,
    loader: ({ params }) => fetchUser(params.id),
  },
]);

function UserProfile() {
  const user = useLoaderData();
  return <p>{user.name}</p>;
}
```

## Key Takeaways

- Routes map URL paths to components; `useParams`/`useSearchParams` read the URL, `useNavigate`/`Link` change it.
- Use `Outlet` for shared layouts (nav bars, sidebars) across nested routes.
- Guard authenticated routes with a wrapper component rather than checking auth inside every page.
