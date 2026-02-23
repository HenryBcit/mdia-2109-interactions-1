# The React Render Process: Variables, State & Maps

Understanding *when* and *why* React re-renders is the key to writing components that are both correct and performant.

---

## 1. What Is a Render?

Every time React renders a component, it simply **calls the component function**. Whatever JSX that function returns becomes what's shown on screen.

```jsx
export default function Greeting() {
  console.log('Greeting rendered!'); // logs every time React calls this function

  return <h1>Hello, world!</h1>;
}
```

React calls (renders) your component:
- On **first mount** — when the component appears on screen for the first time.
- When its **state changes** — via `useState` setter.
- When its **props change** — parent passed new values down.
- When its **parent re-renders** — even if props didn't change.

---

## 2. Regular Variables vs. State

This is the most important distinction in React. Regular variables and state behave very differently during a render.

### Regular Variables — Don't Trigger Re-renders

```jsx
export default function Counter() {
  let count = 0; // reset to 0 on every render

  function handleClick() {
    count = count + 1;
    console.log(count); // logs correctly, but...
  }

  return (
    <div>
      <p>Count: {count}</p>         {/* always shows 0 */}
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

The problem: `count` is just a local variable. When you change it, React doesn't know — so it never re-renders, and the UI never updates.

### State Variables — Trigger Re-renders

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0); // persisted between renders by React

  function handleClick() {
    setCount(count + 1); // tells React: "re-render with new count"
  }

  return (
    <div>
      <p>Count: {count}</p>         {/* updates correctly */}
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

When you call `setCount`, React schedules a re-render. On the next render, `useState` returns the new value.

### The Render Snapshot

Every render is a **snapshot in time**. The values of state captured inside a render are frozen for that render cycle.

```jsx
import { useState } from 'react';

export default function DelayedCounter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);

    setTimeout(() => {
      // ⚠️ This logs the count from *this* render, not the future render
      console.log(count); // logs 0, even after clicking
    }, 3000);
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

Fix this with the **functional updater form** — it always receives the latest state:

```jsx
setCount((prev) => prev + 1); // ✅ always uses the most recent value
```

---

## 3. Variables Computed During Render

Regular variables are perfectly useful for values **derived from** props or state. They're recalculated fresh on every render — which is exactly what you want.

```jsx
export default function OrderSummary({ items, taxRate = 0.1 }) {
  // These are recalculated every render — no useState needed
  const subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0);
  const tax = subtotal * taxRate;
  const total = subtotal + tax;

  const hasItems = items.length > 0;
  const itemCount = items.reduce((sum, item) => sum + item.qty, 0);

  return (
    <div>
      <p>Items: {itemCount}</p>
      <p>Subtotal: ${subtotal.toFixed(2)}</p>
      <p>Tax: ${tax.toFixed(2)}</p>
      <p><strong>Total: ${total.toFixed(2)}</strong></p>
      {!hasItems && <p>Your cart is empty.</p>}
    </div>
  );
}
```

> **Rule of thumb:** If a value can be calculated from props or state, don't put it in `useState`. Just compute it as a variable during render.

---

## 4. Rendering Lists with `.map()`

`.map()` is the standard way to render a list of items in React. It transforms an array of data into an array of JSX elements.

### Basic Map

```jsx
export default function FruitList() {
  const fruits = ['Apple', 'Banana', 'Cherry'];

  return (
    <ul>
      {fruits.map((fruit) => (
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  );
}
```

### The `key` Prop

Every element in a mapped list **must** have a `key`. React uses it to track which items changed, were added, or removed between renders.

```jsx
// ✅ Use a stable unique ID from your data
posts.map((post) => <PostCard key={post.id} post={post} />)

// ⚠️ Using index works but can cause bugs when list order changes
posts.map((post, index) => <PostCard key={index} post={post} />)

// ❌ Never use random values — new key every render = React remounts the component
posts.map((post) => <PostCard key={Math.random()} post={post} />)
```

### Mapping Real Data from State

```jsx
import { useState } from 'react';

export default function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Buy groceries', done: false },
    { id: 2, text: 'Walk the dog', done: true },
    { id: 3, text: 'Write code', done: false },
  ]);

  function toggleTodo(id) {
    setTodos((prev) =>
      prev.map((todo) =>
        todo.id === id ? { ...todo, done: !todo.done } : todo
      )
    );
  }

  return (
    <ul>
      {todos.map((todo) => (
        <li
          key={todo.id}
          style={{ textDecoration: todo.done ? 'line-through' : 'none' }}
          onClick={() => toggleTodo(todo.id)}
        >
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

Notice: `toggleTodo` uses `.map()` on the state array to produce a **new array** with one item changed — it never mutates the original.

---

## 5. Conditional Rendering

React only renders what you return. You control what appears using JavaScript expressions inside JSX.

### Ternary — if/else

```jsx
export default function AuthStatus({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? (
        <p>Welcome back!</p>
      ) : (
        <p>Please log in.</p>
      )}
    </div>
  );
}
```

### `&&` — render only if true

```jsx
export default function Notification({ hasMessages, count }) {
  return (
    <div>
      <h1>Inbox</h1>
      {hasMessages && <span className="badge">{count} new</span>}
    </div>
  );
}
```

> ⚠️ Avoid `{count && <Badge />}` when `count` could be `0` — React renders `0` as text. Use `{count > 0 && <Badge />}` instead.

### Computed variable — complex conditions

For readability, resolve complex conditions into a variable before the return.

```jsx
export default function StatusBanner({ status, isAdmin }) {
  let message;

  if (status === 'loading') message = <p>Loading...</p>;
  else if (status === 'error') message = <p className="error">Something went wrong.</p>;
  else if (isAdmin) message = <p className="admin">Welcome, Admin.</p>;
  else message = <p>Welcome!</p>;

  return <div className="banner">{message}</div>;
}
```

---

## 6. The Full Render Flow — All Together

Here's a complete component showing how variables, state, `.map()`, and conditional rendering work together during each render cycle:

```jsx
import { useState, useEffect } from 'react';

export default function ProductGrid({ category }) {
  const [products, setProducts] = useState([]);
  const [search, setSearch] = useState('');
  const [loading, setLoading] = useState(true);

  // 1. Fetch data when category changes
  useEffect(() => {
    setLoading(true);

    fetch(`/api/products?category=${category}`)
      .then((res) => res.json())
      .then((data) => {
        setProducts(data);
        setLoading(false);
      });
  }, [category]);

  // 2. Computed variables — derived from state, recalculated each render
  const filtered = products.filter((p) =>
    p.name.toLowerCase().includes(search.toLowerCase())
  );

  const hasResults = filtered.length > 0;
  const totalCount = products.length;

  // 3. Return JSX — what React will display this render
  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search products..."
      />
      <p>{totalCount} products — {filtered.length} shown</p>

      {loading ? (
        <p>Loading...</p>
      ) : !hasResults ? (
        <p>No products match "{search}".</p>
      ) : (
        <div className="grid">
          {filtered.map((product) => (
            <div key={product.id} className="product-card">
              <h3>{product.name}</h3>
              <p>${product.price.toFixed(2)}</p>
              {product.inStock ? (
                <span className="badge green">In Stock</span>
              ) : (
                <span className="badge red">Out of Stock</span>
              )}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

**What happens on each render:**
1. React calls `ProductGrid` as a function.
2. `useState` returns the current values of `products`, `search`, and `loading`.
3. `filtered`, `hasResults`, and `totalCount` are recalculated fresh from the current state.
4. React evaluates the JSX and figures out what changed since last render (this is called **reconciliation**).
5. React updates only the parts of the DOM that actually changed.

---

## Quick Reference

| Concept                    | Triggers Re-render? | Persists Between Renders? | Use When                                      |
|----------------------------|---------------------|---------------------------|-----------------------------------------------|
| `let` / `const` variable   | ❌ No               | ❌ No (reset each render) | Derived values, computed from state/props     |
| `useState`                 | ✅ Yes              | ✅ Yes                    | Any data the UI depends on that changes       |
| `.map()` in JSX            | —                   | —                         | Rendering arrays of data as lists             |
| `key` prop                 | —                   | —                         | Required on every mapped element; use stable IDs |