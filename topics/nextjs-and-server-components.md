# Next.js and React Server Components

## Why Next.js

Next.js is a React framework that adds file-based routing, server-side rendering (SSR), static site generation (SSG), API routes, and bundling/optimization on top of React — most production React apps use a framework like this rather than assembling their own build from scratch.

## Rendering Strategies

| Strategy | When HTML is generated | Good for |
|-----------|--------------------------|-----------|
| **CSR** (Client-Side Rendering) | In the browser, after JS loads | Highly interactive, behind-login apps |
| **SSR** (Server-Side Rendering) | Per-request, on the server | Pages needing fresh data + SEO |
| **SSG** (Static Site Generation) | At build time | Content that rarely changes (blogs, docs, marketing pages) |
| **ISR** (Incremental Static Regeneration) | At build time, then re-generated periodically | Mostly-static pages with occasional updates |

## File-Based Routing (App Router)

```
app/
  page.tsx            -> /
  about/page.tsx       -> /about
  blog/[slug]/page.tsx  -> /blog/:slug
  dashboard/layout.tsx  -> shared layout for everything under /dashboard
```

```jsx
// app/blog/[slug]/page.tsx
export default function BlogPost({ params }) {
  return <h1>Post: {params.slug}</h1>;
}
```

## React Server Components (RSC)

With the App Router, components are **Server Components by default** — they render on the server and send only HTML/serialized output to the browser, with zero JS shipped for that component. This differs fundamentally from traditional SSR, which still hydrates the whole tree with client-side JS.

```jsx
// Server Component (default, no directive needed)
async function ProductList() {
  const products = await db.products.findMany(); // runs on the server only
  return (
    <ul>
      {products.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

Benefits: smaller client JS bundles, direct backend/database access without an API layer, secrets never reach the browser.

## Client Components

Add `"use client"` at the top of a file when a component needs interactivity (state, effects, event handlers, browser-only APIs):

```jsx
"use client";

function LikeButton() {
  const [liked, setLiked] = useState(false);
  return <button onClick={() => setLiked(!liked)}>{liked ? "Liked" : "Like"}</button>;
}
```

Server and Client Components can be composed together — a Server Component can render a Client Component, passing serializable props to it.

## Data Fetching

Server Components can `await` data directly in the component body — no `useEffect`/loading-state dance needed for the initial render:

```jsx
async function UserProfile({ userId }) {
  const user = await fetch(`https://api.example.com/users/${userId}`).then(r => r.json());
  return <h1>{user.name}</h1>;
}
```

## API Routes

```jsx
// app/api/hello/route.ts
export async function GET() {
  return Response.json({ message: 'Hello' });
}
```

## Key Takeaways

- Choose a rendering strategy per page based on how often the data changes and whether SEO matters.
- Server Components reduce client JS and simplify data fetching, but need `"use client"` for anything interactive.
- Next.js (or a similar framework like Remix) is the standard way most teams ship production React today rather than hand-rolling routing/SSR.
