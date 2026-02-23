# React Components: useState, useEffect & Props

React components are JavaScript functions that return JSX. They accept **props** for input, use **useState** to manage local data, and use **useEffect** to sync with the outside world.

---

## 1. Props — Passing Data Into Components

Props are how a parent component passes data **down** to a child. They're read-only — a child never modifies its own props.

```jsx
// components/UserCard.jsx
export default function UserCard({ name, age, isAdmin }) {
  return (
    <div className="card">
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <span className="badge">Admin</span>}
    </div>
  );
}
```

```jsx
// app/page.jsx
import UserCard from '@/components/UserCard';

export default function Page() {
  return (
    <UserCard
      name="Alice"
      age={30}
      isAdmin={true}
    />
  );
}
```

### Default Prop Values

```jsx
export default function Button({ label = 'Click me', variant = 'primary' }) {
  return <button className={`btn btn-${variant}`}>{label}</button>;
}
```

### The `children` Prop

`children` is a special prop — it's whatever you put **between** a component's opening and closing tags.

```jsx
// components/Card.jsx
export default function Card({ title, children }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-body">{children}</div>
    </div>
  );
}
```

```jsx
// usage
<Card title="Welcome">
  <p>This is the card body.</p>
  <button>Learn more</button>
</Card>
```

---

## 2. useState — Managing Local State

`useState` lets a component remember values between renders. When state changes, React re-renders the component.

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0); // initial value = 0

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

`useState` returns a pair: the **current value** and a **setter function**. Always use the setter — never mutate state directly.

### Multiple State Variables

```jsx
import { useState } from 'react';

export default function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState(null);

  function handleSubmit(e) {
    e.preventDefault();
    if (!email || !password) {
      setError('All fields required.');
      return;
    }
    setError(null);
    console.log('Submitting:', { email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" />
      <input value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" type="password" />
      {error && <p className="error">{error}</p>}
      <button type="submit">Log In</button>
    </form>
  );
}
```

### State with Objects

When state is an object, always spread the previous state to avoid losing other fields.

```jsx
const [user, setUser] = useState({ name: '', age: 0, city: '' });

// ✅ Correct — spread old state, then override the field
setUser((prev) => ({ ...prev, name: 'Alice' }));

// ❌ Wrong — this wipes out age and city
setUser({ name: 'Alice' });
```

---

## 3. useEffect — Syncing with the Outside World

`useEffect` runs **after** the component renders. Use it for: fetching data, setting up subscriptions, timers, or directly touching the DOM.

```jsx
import { useEffect } from 'react';

useEffect(() => {
  // runs after every render (⚠️ usually not what you want)
});

useEffect(() => {
  // runs once — on mount only
}, []);

useEffect(() => {
  // runs when `userId` changes
}, [userId]);
```

### Fetching Data on Mount

```jsx
import { useState, useEffect } from 'react';

export default function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);

    fetch(`https://api.example.com/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        setUser(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]); // re-fetch whenever userId changes

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### Cleanup — Avoiding Memory Leaks

Return a function from `useEffect` to clean up when the component unmounts or before the effect runs again.

```jsx
useEffect(() => {
  const interval = setInterval(() => {
    console.log('tick');
  }, 1000);

  return () => clearInterval(interval); // cleanup on unmount
}, []);
```

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch('/api/data', { signal: controller.signal })
    .then((res) => res.json())
    .then(setData);

  return () => controller.abort(); // cancel fetch if component unmounts
}, []);
```

---

## 4. Putting It All Together

A realistic component using props, useState, and useEffect together:

```jsx
// components/PostList.jsx
import { useState, useEffect } from 'react';
import PostCard from './PostCard';

export default function PostList({ authorId, limit = 5 }) {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [page, setPage] = useState(1);

  useEffect(() => {
    setLoading(true);

    fetch(`/api/posts?author=${authorId}&page=${page}&limit=${limit}`)
      .then((res) => res.json())
      .then((data) => {
        setPosts(data);
        setLoading(false);
      });
  }, [authorId, page, limit]); // re-run if any of these change

  return (
    <div>
      {loading ? (
        <p>Loading posts...</p>
      ) : (
        posts.map((post) => <PostCard key={post.id} post={post} />)
      )}

      <div className="pagination">
        <button onClick={() => setPage((p) => p - 1)} disabled={page === 1}>
          Previous
        </button>
        <span>Page {page}</span>
        <button onClick={() => setPage((p) => p + 1)}>Next</button>
      </div>
    </div>
  );
}
```

```jsx
// components/PostCard.jsx
export default function PostCard({ post }) {
  const { title, excerpt, date } = post; // destructure props

  return (
    <article className="post-card">
      <h3>{title}</h3>
      <p>{excerpt}</p>
      <time>{new Date(date).toLocaleDateString()}</time>
    </article>
  );
}
```

```jsx
// app/page.jsx
import PostList from '@/components/PostList';

export default function Page() {
  return <PostList authorId="42" limit={10} />;
}
```

---

## Quick Reference

| Hook / Concept    | Purpose                                          | Key Rule                                              |
|-------------------|--------------------------------------------------|-------------------------------------------------------|
| **Props**         | Pass data from parent → child                    | Read-only — never modify props directly               |
| **`children`**    | Pass JSX between component tags                  | Always available as `props.children`                  |
| **`useState`**    | Store and update local component data            | Always use the setter function, never mutate directly |
| **`useEffect`**   | Run code after render (fetch, timers, listeners) | Return a cleanup function to avoid memory leaks       |
| **Dependency array `[]`** | Controls when `useEffect` re-runs      | Empty `[]` = mount only; `[val]` = when val changes  |

