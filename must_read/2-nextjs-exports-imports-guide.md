# Exports & Imports in Next.js

Next.js is built on React and uses **ES Modules** throughout. Here's how exports and imports work in a real Next.js project.

---

## Project Structure

```
my-app/
├── app/
│   └── page.jsx          ← default export (the page)
├── components/
│   └── Button.jsx        ← default export (the component)
├── lib/
│   └── utils.js          ← named exports (helper functions)
└── constants/
    └── config.js         ← named exports (config values)
```

---

## 1. Named Exports — utility/helper files

Use named exports when a file provides **multiple utilities**.

```js
// lib/utils.js
export function formatDate(date) {
  return new Date(date).toLocaleDateString();
}

export function truncate(str, length = 100) {
  return str.length > length ? str.slice(0, length) + '...' : str;
}

export const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
```

```jsx
// app/page.jsx
import { formatDate, truncate } from '@/lib/utils';

export default function HomePage() {
  return (
    <div>
      <p>{formatDate('2024-01-15')}</p>
      <p>{truncate('A very long piece of text...', 20)}</p>
    </div>
  );
}
```

> `@/` is a Next.js path alias for the project root — no more `../../..`.

---

## 2. Default Export — components

Use `export default` for **components** — each component file typically exports one thing.

```jsx
// components/Button.jsx
export default function Button({ children, onClick }) {
  return (
    <button onClick={onClick} className="btn">
      {children}
    </button>
  );
}
```

```jsx
// app/page.jsx
import Button from '@/components/Button';  // any name works

export default function HomePage() {
  return <Button onClick={() => alert('clicked!')}>Click me</Button>;
}
```

---

## 3. Mixing Default + Named Exports — components with variants

```jsx
// components/Card.jsx

// Named exports — specific card variants
export function CardHeader({ title }) {
  return <h2 className="card-header">{title}</h2>;
}

export function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

// Default export — the main Card component
export default function Card({ title, children }) {
  return (
    <div className="card">
      <CardHeader title={title} />
      <CardBody>{children}</CardBody>
    </div>
  );
}
```

```jsx
// app/page.jsx
import Card, { CardHeader, CardBody } from '@/components/Card';

export default function HomePage() {
  return (
    <>
      {/* Use the full Card */}
      <Card title="Hello">Some content here</Card>

      {/* Or use parts individually */}
      <div>
        <CardHeader title="Custom Layout" />
        <CardBody>Custom content</CardBody>
      </div>
    </>
  );
}
```

---

## 4. Default Export — Next.js Pages & Layouts

Next.js **requires** a `default export` from every `page.jsx` and `layout.jsx`. This is how Next.js knows what to render.

```jsx
// app/layout.jsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

```jsx
// app/about/page.jsx
export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

> ⚠️ Forgetting `export default` on a page will cause a Next.js build error.

---

## 5. Named Exports — Next.js Route Handlers & Metadata

For API routes and metadata, Next.js uses **named exports** with specific names it looks for.

```js
// app/api/hello/route.js
export async function GET(request) {
  return Response.json({ message: 'Hello!' });
}

export async function POST(request) {
  const body = await request.json();
  return Response.json({ received: body });
}
```

```jsx
// app/page.jsx — metadata uses a named export too
export const metadata = {
  title: 'My App',
  description: 'Welcome to my Next.js app',
};

export default function HomePage() {
  return <h1>Home</h1>;
}
```

---

## Quick Reference

| What you're exporting        | Use                  | Example                          |
|------------------------------|----------------------|----------------------------------|
| A React component (main)     | `export default`     | `Button.jsx`, `Card.jsx`         |
| A Next.js page or layout     | `export default`     | `page.jsx`, `layout.jsx`         |
| Multiple utility functions   | Named `export`       | `lib/utils.js`                   |
| API route handlers           | Named `export`       | `route.js` (`GET`, `POST`, etc.) |
| Page metadata                | Named `export`       | `export const metadata = {...}`  |
| Config constants             | Named `export`       | `constants/config.js`            |
