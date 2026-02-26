
---

React by itself is only a UI library it helps you build components, but it doesn’t tell you _how_ to structure an app, handle routing, fetch data efficiently, optimize performance, or prepare an app for production. Next.js fills this gap. It provides a complete, opinionated framework on top of React so developers can focus on building features instead of wiring infrastructure.

The **main purpose of Next.js** is to make web applications **fast, scalable, and production-ready by default**. Traditional React apps (like those created with CRA) run mostly on the client. The browser downloads a large JavaScript bundle, then renders the page. This can lead to slow initial load times, poor SEO, and bad performance on low-end devices.

One of the key reasons **Next.js exists is rendering flexibility**. It supports **Server-Side Rendering (SSR)**, **Static Site Generation (SSG)**, **Incremental Static Regeneration (ISR)**, and **Client-Side Rendering (CSR)** all within the same application. This means you can render pages on the server for SEO and performance, pre-generate pages at build time for speed, or render dynamically on the client when needed.

Next.js also exists to **simplify routing and application structure**. Instead of configuring routing libraries manually, Next.js uses a **file-based routing system**. Files and folders automatically become routes. This reduces boilerplate, enforces consistency, and makes large applications easier to reason about. Next.js also exists to **blur the line between frontend and backend**. With **API routes and server actions**, developers can write backend logic directly inside the same project.

## Tutorial

Easiest way to create next application is `npx create-next-app@latest`. It will create ready to run project with default configuration installing tailwind,eslint,app router etc. This is important because Next.js encourages **convention over configuration** the folder structure itself defines how your app works.

### Routing

React is convention over configuration entire routing system is dictated by the file system. This file system is living inside the `app/` directory. `Folder` hierarchy defines the routes while file name `page.jsx` automaitcally defines the actual page inside page.For example a folder `/blogs` may contain file name `/page.tsx`. In the home `/` will be defined by the file `page.tsx` in the app folder. All this is part of `AppRouter` feature of next. 

Unlike AppRouter there exists another way of routing - The **Pages Router** is built around the idea that every file inside the `pages/` directory automatically becomes a route. For example, `pages/index.js` maps to `/` and `pages/blog/[id].js` maps to a dynamic route like `/blog/123`.Data fetching in the Pages Router is done using special functions like `getStaticProps`, `getServerSideProps`, and `getStaticPaths`, which run outside the React component itself. While this separation works well, it can sometimes feel indirect because the data-fetching logic is detached from the component tree.

The **App Router**, introduced with the `app/` directory, is designed around React Server Components and embraces a more component-centric way of thinking. Instead of routes being just files, routes are built from nested folders that naturally mirror UI structure. A `page.js` file defines a route, while `layout.js` files allow you to create persistent layouts that wrap child routes without reloading. This makes it much easier to model complex interfaces where parts of the page, like navigation bars or side panels, should remain consistent while only the main content changes.

One of the biggest conceptual differences lies in **data fetching**. In the App Router, data fetching happens directly inside components using `fetch` and async functions, and by default these components run on the server. This removes the need for separate lifecycle-like methods such as `getServerSideProps`. As a result, data, UI, and routing logic feel more closely connected, which improves readability and maintainability. It also allows finer-grained control over caching, revalidation, and streaming responses.

Another key difference is how **rendering behavior** is handled. The Pages Router primarily relies on client-side React components with server-side rendering as an optional enhancement. In contrast, the App Router defaults to server components and only sends JavaScript to the browser when necessary.When client-side interactivity is needed, you explicitly opt in by using the `"use client"` directive, making the boundary between server and client logic clear and intentional.

### Anatomy of pages

Each `page.js` file exports a React component, and that component becomes a page users can visit. Unlike traditional React apps, pages in Next.js are **server components by default**, meaning they run on the server and send HTML to the browser. Page is the entry point of a route whenever a user navigates to a URL, Next.js looks for a corresponding `page.js` file and renders it.

At its simplest level, a `page.js` file **must export a default React component**. That component represents the UI for the route. For example, a `page.js` inside `app/blog` will render when a user visits `/blog`. This default export is mandatory — without it, the route does not exist.

```jsx
export default function Page() {
  return <h1>Welcome to Home Page</h1>
}
```


One of the most important aspects of `page.js` is that it is **a Server Component by default**. This means the code runs on the server unless you explicitly opt into client-side behavior using the `"use client"` directive at the top of the file. Because of this, you can directly fetch data from databases, call internal APIs, or access secrets without worrying about exposing them to the browser. This server-first nature is a key design change compared to the older Pages Router.

```jsx
"use client"

import { useState } from "react"

export default function Page() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  )
}

```

The function exported from `page.js` receives a special set of **props**, mainly `params` and `searchParams`. The `params` object contains values from dynamic route segments, such as `[id]`, while `searchParams` contains query string values from the URL. These props allow the page to respond directly to the structure of the URL without additional routing logic.

```
app/
 └─ blog/
     └─ [id]/
         └─ page.js   → /blog/123
```

```jsx
export default function Page({ params }) {
  return <h1>Blog ID: {params.id}</h1>
}
```

Similarly query params

```jsx
export default function Page({ searchParams }) {
  return (
    <>
      <p>Category: {searchParams.category}</p>
      <p>Sort: {searchParams.sort}</p>
    </>
  )
}
```

Another part of the anatomy is **data fetching**. Since `page.js` is a Server Component, you can fetch data directly inside the component using `fetch`, and Next.js automatically optimizes it through caching, streaming, and revalidation. You don’t need special lifecycle methods like `getServerSideProps` or `getStaticProps`; instead, data fetching becomes a natural part of the component’s execution.`page.js` can also export **route-level metadata**. By exporting a `metadata` object or a  `generateMetadata` function, you define things like the page title, description, Open Graph tags, and more.

```jsx
export default async function Page() {
  const data = await fetch("https://api.example.com/posts").then(res => res.json())

  return (
    <ul>
      {data.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

Static data can be cached forever. 

```js
await fetch("https://api.example.com/posts")
```

While we can revalidate for dynamic data

```js
await fetch("https://api.example.com/posts", {
  next: { revalidate: 60 }
})
```

We can also opt to not cache

```jsx
await fetch("https://api.example.com/posts", {
  cache: "no-store"
})
```

We can also control this at page level 
```jsx
export const dynamic = "force-dynamic"
```

Rendering behavior is another key element. A page can be **static, dynamic, or hybrid**, and this is controlled using exports like `dynamic`, `revalidate`, or by how data is fetched. For example, a fully static page can be generated at build time, while a dynamic page can be rendered on every request. This control is part of the page’s anatomy because it defines _when_ and _how often_ the page is rendered.

We can also export special framework recognised exports, and Next.js autmomatically configures the routes behaviour on the basis of it.  They are **not regular JavaScript constants** in the usual sense; they act more like **declarative configuration hooks** that Next.js reads at build time and request time.

Example

```js
export const dynamic = "force-static"
export const revalidate = 60
export const fetchCache = "force-no-store"
```

Important exports are

| Export             | Purpose                     |
| ------------------ | --------------------------- |
| `dynamic`          | Static vs Dynamic rendering |
| `revalidate`       | ISR timing                  |
| `fetchCache`       | Default fetch caching       |
| `runtime`          | Node vs Edge                |
| `preferredRegion`  | Deployment region           |
| `dynamicParams`    | Dynamic route fallback      |
| `metadata`         | Static SEO                  |
| `generateMetadata` | Dynamic SEO                 |

Finally, `page.js` participates in **layout composition**. It does not exist in isolation it is rendered inside the nearest `layout.js` file above it in the directory tree. This allows shared UI like headers or sidebars to wrap the page automatically, without the page needing to know about them explicitly.

### ISR

**ISR (Incremental Static Regeneration)** is a rendering strategy in Next.js that sits **between fully static pages and fully dynamic (SSR) pages**. It allows you to serve **fast, static HTML** while still keeping the content **fresh over time**, without rebuilding the entire application. 

Incremental means pages are generated individually and only expired pages are generated. IT solves the problem of Static pages which are generated once and content may become stale. Whereas with server side rendering page is always rendered on every request. ISR regenretes the pages on intervals. 

#### How ISR Works (Mental Model)

1. Page is **generated at build time** and cached as static HTML
2. Users are served this cached page instantly
3. After a specified time (`revalidate`), the page becomes **stale**
4. The **next request** triggers regeneration in the background
5. Users still get the **old page**
6. Once regeneration finishes, the **new page replaces the old one**

ISR should not be used when the content is client specific.  Content is user-specific (dashboards, profiles)

