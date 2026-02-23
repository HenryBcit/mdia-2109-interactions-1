# Conditional Rendering in React — Every Pattern Explained

React conditional rendering is just JavaScript. Since JSX is an expression, you can use any JS technique to decide what gets rendered. Here's every pattern, when to use it, and when to avoid it.

---

## 1. `if` / `else` — Before the Return

The most readable approach for complex conditions. Compute what to render in a variable, then use it in the JSX.

```jsx
export default function Dashboard({ role, isLoggedIn }) {
  if (!isLoggedIn) {
    return <LoginPage />;  // early return — nothing else renders
  }

  let content;

  if (role === 'admin') {
    content = <AdminPanel />;
  } else if (role === 'editor') {
    content = <EditorPanel />;
  } else {
    content = <UserPanel />;
  }

  return (
    <div className="dashboard">
      <Navbar />
      {content}
    </div>
  );
}
```

**Best for:** Multiple branches, complex logic, or when you want to keep JSX clean.  
**Avoid when:** The logic is simple — a ternary is more concise.

---

## 2. Early Return — Guard Clauses

Return early from the component to handle edge cases before the main render. This keeps the happy path clean and avoids deeply nested conditionals.

```jsx
export default function UserProfile({ user, loading, error }) {
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return null; // render nothing

  // if we get here, user is guaranteed to exist
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**Best for:** Loading states, error states, empty/null guards at the top of a component.  
**Avoid when:** You only need to conditionally render a small part of the JSX — use inline patterns instead.

---

## 3. Ternary Operator `? :` — Inline if/else

The go-to for inline conditions with two branches. Lives directly inside JSX.

```jsx
export default function SubscribeButton({ isSubscribed }) {
  return (
    <button className={isSubscribed ? 'btn-secondary' : 'btn-primary'}>
      {isSubscribed ? 'Unsubscribe' : 'Subscribe'}
    </button>
  );
}
```

Ternaries can also swap entire components:

```jsx
export default function AuthGate({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? (
        <UserDashboard />
      ) : (
        <LoginPrompt />
      )}
    </div>
  );
}
```

**Best for:** Simple two-branch conditions inline in JSX.  
**Avoid when:** Logic is complex or nested — deeply nested ternaries become unreadable fast.

```jsx
// ❌ Hard to read — use if/else variables instead
{isAdmin ? isVerified ? <AdminPanel /> : <VerifyPrompt /> : <UserPanel />}

// ✅ Much clearer
let panel;
if (isAdmin && isVerified) panel = <AdminPanel />;
else if (isAdmin) panel = <VerifyPrompt />;
else panel = <UserPanel />;
```

---

## 4. `&&` — Render Only If True

Renders something when a condition is true, nothing when false. The right side only evaluates if the left side is truthy.

```jsx
export default function Inbox({ messages, isAdmin }) {
  return (
    <div>
      <h1>Inbox</h1>
      {messages.length > 0 && <MessageList messages={messages} />}
      {isAdmin && <AdminToolbar />}
    </div>
  );
}
```

### ⚠️ The Zero Gotcha

If your condition is a number, `&&` will render `0` as text when the value is falsy — because `0` is falsy but React renders it as the character `"0"`.

```jsx
// ❌ Renders "0" on screen when count is 0
{count && <Badge count={count} />}

// ✅ Use an explicit boolean comparison
{count > 0 && <Badge count={count} />}

// ✅ Or convert to boolean
{!!count && <Badge count={count} />}
```

**Best for:** Optional UI elements that should simply disappear when not needed.  
**Avoid when:** Your condition could be `0`, `NaN`, or other falsy non-boolean values.

---

## 5. `||` — Fallback / Default Value

Renders the right side when the left side is falsy. Good for fallbacks and default content.

```jsx
export default function Avatar({ name, imageUrl }) {
  return (
    <div>
      <img src={imageUrl || '/default-avatar.png'} alt={name} />
      <p>{name || 'Anonymous'}</p>
    </div>
  );
}
```

**Best for:** Providing fallback values for optional props.  
**Avoid when:** The left side could be intentionally `0` or `false` — `||` will skip those too. Use `??` instead.

---

## 6. Nullish Coalescing `??` — Null/Undefined Fallback Only

Like `||` but only falls back for `null` or `undefined`, not for `0` or `false`.

```jsx
export default function ScoreDisplay({ score }) {
  return (
    <div>
      {/* || would show "No score" for score=0, which is a valid score */}
      <p>Score: {score ?? 'No score yet'}</p>
    </div>
  );
}
```

```jsx
// Comparison
score = 0;
score || 'No score yet'  // → 'No score yet'  ❌ wrong, 0 is valid
score ?? 'No score yet'  // → 0               ✅ correct
```

**Best for:** Fallbacks where `0`, `false`, or empty string are valid values you want to preserve.

---

## 7. `null` — Render Nothing

Returning `null` from a component (or in JSX) renders absolutely nothing — no empty tags, no whitespace.

```jsx
export default function Banner({ show, message }) {
  if (!show) return null;

  return <div className="banner">{message}</div>;
}
```

```jsx
// Also works inline
export default function Page({ isProUser }) {
  return (
    <div>
      <h1>Welcome</h1>
      {isProUser ? <ProFeatures /> : null}
    </div>
  );
}
```

**Best for:** Completely hiding a component based on a condition. The component still mounts/unmounts, which triggers `useEffect` cleanup.

---

## 8. Switch Statements — Many Branches

For more than 3-4 branches, a `switch` keeps things cleaner than chained `if/else`.

```jsx
export default function StatusBadge({ status }) {
  let label, className;

  switch (status) {
    case 'active':
      label = 'Active';
      className = 'badge-green';
      break;
    case 'pending':
      label = 'Pending';
      className = 'badge-yellow';
      break;
    case 'suspended':
      label = 'Suspended';
      className = 'badge-red';
      break;
    case 'expired':
      label = 'Expired';
      className = 'badge-gray';
      break;
    default:
      label = 'Unknown';
      className = 'badge-gray';
  }

  return <span className={`badge ${className}`}>{label}</span>;
}
```

**Best for:** Rendering different UI based on a single value with many possible states (status, role, step, etc.).

---

## 9. Object Lookup Map — Declarative Switch Alternative

Instead of `switch`, map values to components or JSX in a plain object. More declarative and often cleaner.

```jsx
const STEP_COMPONENTS = {
  personal: <PersonalInfoStep />,
  payment:  <PaymentStep />,
  review:   <ReviewStep />,
  confirm:  <ConfirmStep />,
};

export default function Wizard({ currentStep }) {
  return (
    <div className="wizard">
      {STEP_COMPONENTS[currentStep] ?? <p>Unknown step.</p>}
    </div>
  );
}
```

With dynamic props, use functions instead of static JSX:

```jsx
const STATUS_BADGE = {
  active:    (label) => <span className="badge-green">{label}</span>,
  pending:   (label) => <span className="badge-yellow">{label}</span>,
  suspended: (label) => <span className="badge-red">{label}</span>,
};

export default function StatusBadge({ status, label }) {
  const render = STATUS_BADGE[status];
  return render ? render(label) : <span className="badge-gray">Unknown</span>;
}
```

**Best for:** Many variants driven by a single string key. Cleaner than switch, easy to extend.  
**Avoid when:** Conditions involve complex boolean logic — object lookup only works for exact key matches.

---

## 10. Immediately Invoked Function Expression (IIFE)

Lets you use full `if/else` blocks inline inside JSX without extracting a variable. Useful but use sparingly.

```jsx
export default function ComplexCard({ type, data, isLoading }) {
  return (
    <div>
      {(() => {
        if (isLoading) return <Spinner />;
        if (type === 'chart') return <ChartView data={data} />;
        if (type === 'table') return <TableView data={data} />;
        return <DefaultView data={data} />;
      })()}
    </div>
  );
}
```

**Best for:** When you need multi-branch if/else inline but don't want a separate variable.  
**Avoid when:** It makes JSX harder to read — a pre-computed variable is often cleaner.

---

## 11. Component Composition — Conditional Rendering via Props

Sometimes the cleanest approach is to delegate the decision to a wrapper component.

```jsx
// A generic Guard component
function Show({ when, fallback = null, children }) {
  return when ? children : fallback;
}

export default function Page({ isLoggedIn, isAdmin }) {
  return (
    <div>
      <Show when={isLoggedIn} fallback={<LoginPrompt />}>
        <Dashboard />
      </Show>

      <Show when={isAdmin}>
        <AdminPanel />
      </Show>
    </div>
  );
}
```

**Best for:** Reusable conditional logic across many components. Keeps JSX declarative and readable.

---

## All Patterns at a Glance

```jsx
import { useState } from 'react';

export default function AllPatterns({ user, role, score, status, step }) {
  const [show, setShow] = useState(true);

  // 1. Early return — guard clause
  if (!user) return <p>No user found.</p>;

  // 2. if/else variable — complex branch
  let panel;
  if (role === 'admin') panel = <AdminPanel />;
  else if (role === 'editor') panel = <EditorPanel />;
  else panel = <UserPanel />;

  // 3. Object lookup — status badge
  const badges = {
    active: <span className="green">Active</span>,
    pending: <span className="yellow">Pending</span>,
    expired: <span className="gray">Expired</span>,
  };

  return (
    <div>
      {/* 4. Ternary — two branches */}
      {user.isVerified ? <VerifiedBadge /> : <UnverifiedBanner />}

      {/* 5. && — render only if true */}
      {user.isAdmin && <AdminToolbar />}

      {/* 6. ?? — null/undefined fallback */}
      <p>Score: {score ?? 'Not rated yet'}</p>

      {/* 7. null — hide explicitly */}
      {show ? <Notification /> : null}

      {/* 8. if/else variable result */}
      {panel}

      {/* 9. Object lookup */}
      {badges[status] ?? <span className="gray">Unknown</span>}

      {/* 10. IIFE — inline multi-branch */}
      {(() => {
        if (step === 1) return <StepOne />;
        if (step === 2) return <StepTwo />;
        return <StepThree />;
      })()}
    </div>
  );
}
```

---

## Choosing the Right Pattern

| Situation                                      | Best Pattern                         |
|------------------------------------------------|--------------------------------------|
| Loading / error / null guards                  | Early return                         |
| Two branches inline                            | Ternary `? :`                        |
| Show or hide a single element                  | `&&`                                 |
| Fallback for null/undefined                    | `??`                                 |
| Fallback for any falsy value                   | `\|\|`                               |
| 3+ branches based on a single value            | Object lookup or `switch`            |
| Complex boolean logic with many branches       | `if/else` variable before return     |
| Render nothing at all                          | Return `null`                        |
| Reusable conditional logic                     | Wrapper component (`<Show when>`)    |