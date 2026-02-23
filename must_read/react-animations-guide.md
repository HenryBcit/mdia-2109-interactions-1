# Animating React Components — CSS Transitions, Animations, State & Effects

React doesn't have a built-in animation system — instead you combine **CSS** (transitions & keyframes) with **state** and **refs** to drive animations. Here's every pattern, from the simplest fade to complex mount/unmount sequences.

---

## 1. CSS Transition on State Change — The Simplest Pattern

The most common animation pattern: toggle a CSS class via state, let CSS `transition` handle the visual change.

```css
/* styles.css */
.box {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 300ms ease, transform 300ms ease;
}

.box.visible {
  opacity: 1;
  transform: translateY(0);
}
```

```jsx
import { useState } from 'react';
import './styles.css';

export default function FadeInBox() {
  const [visible, setVisible] = useState(false);

  return (
    <div>
      <button onClick={() => setVisible((v) => !v)}>Toggle</button>
      <div className={`box ${visible ? 'visible' : ''}`}>
        Hello, I fade in!
      </div>
    </div>
  );
}
```

**How it works:** React adds/removes the `visible` class. CSS `transition` smoothly interpolates between the two states.

**Best for:** Hover effects, toggles, show/hide panels, tabs.

---

## 2. CSS Keyframe Animation on Mount

CSS `@keyframes` run automatically when an element appears in the DOM. No state needed — just apply the class.

```css
/* styles.css */
@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.alert {
  animation: slideDown 400ms ease forwards;
}
```

```jsx
export default function Alert({ message }) {
  return (
    <div className="alert">
      {message}
    </div>
  );
}

// In parent — conditionally render to trigger animation on mount
export function Page() {
  const [show, setShow] = useState(false);

  return (
    <div>
      <button onClick={() => setShow(true)}>Show Alert</button>
      {show && <Alert message="Action completed!" />}
    </div>
  );
}
```

**How it works:** Every time `<Alert>` mounts (appears in DOM), the keyframe animation runs from the start.

**Best for:** Notifications, toasts, modals, any element that appears and should immediately animate in.

---

## 3. Animate In — Delayed Class Addition with `useEffect`

Sometimes you need to apply a class *after* mount, not during, to trigger a CSS transition. If you set the class immediately on mount, the browser may skip the transition.

```css
.drawer {
  transform: translateX(-100%);
  transition: transform 350ms cubic-bezier(0.4, 0, 0.2, 1);
}

.drawer.open {
  transform: translateX(0);
}
```

```jsx
import { useState, useEffect } from 'react';

export default function Drawer({ isOpen }) {
  const [animate, setAnimate] = useState(false);

  useEffect(() => {
    if (isOpen) {
      // Tiny delay lets the browser paint the initial state first
      // so the transition has a "from" value to start from
      const timer = setTimeout(() => setAnimate(true), 10);
      return () => clearTimeout(timer);
    } else {
      setAnimate(false);
    }
  }, [isOpen]);

  if (!isOpen && !animate) return null;

  return (
    <div className={`drawer ${animate ? 'open' : ''}`}>
      Drawer content here
    </div>
  );
}
```

**How it works:** The component mounts with `drawer` (closed position). After 10ms, `open` is added, and the transition fires from `translateX(-100%)` to `translateX(0)`.

**Best for:** Drawers, sidebars, panels that slide in from off-screen.

---

## 4. Animate Out Before Unmount — The Core Pattern

CSS transitions only work on elements that exist in the DOM. If you unmount a component immediately, there's no time to animate out. The solution: delay the unmount until the animation finishes.

```css
.modal-overlay {
  opacity: 0;
  transition: opacity 250ms ease;
}

.modal-overlay.visible {
  opacity: 1;
}

.modal-content {
  transform: scale(0.9) translateY(10px);
  transition: transform 250ms ease, opacity 250ms ease;
  opacity: 0;
}

.modal-overlay.visible .modal-content {
  transform: scale(1) translateY(0);
  opacity: 1;
}
```

```jsx
import { useState, useEffect } from 'react';

export default function Modal({ isOpen, onClose, children }) {
  const [mounted, setMounted] = useState(false);
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    if (isOpen) {
      setMounted(true);                          // 1. Mount the DOM node
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          setVisible(true);                      // 2. Trigger CSS transition in
        });
      });
    } else {
      setVisible(false);                         // 3. Trigger CSS transition out
      const timer = setTimeout(() => {
        setMounted(false);                       // 4. Unmount after transition
      }, 250);                                   //    250ms = transition duration
      return () => clearTimeout(timer);
    }
  }, [isOpen]);

  if (!mounted) return null;

  return (
    <div className={`modal-overlay ${visible ? 'visible' : ''}`} onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>
  );
}
```

**The sequence:**
1. `isOpen` becomes `true` → component mounts → next frame adds `visible` class → CSS transitions in.
2. `isOpen` becomes `false` → `visible` removed → CSS transitions out → after 250ms, component unmounts.

**Best for:** Modals, dialogs, toasts, dropdowns — anything that needs a smooth enter *and* exit.

---

## 5. `onAnimationEnd` / `onTransitionEnd` — Unmount After Animation

Instead of a hardcoded `setTimeout`, listen for when the CSS animation or transition actually finishes.

```css
@keyframes fadeOut {
  from { opacity: 1; transform: scale(1); }
  to   { opacity: 0; transform: scale(0.8); }
}

.toast {
  animation: slideIn 300ms ease forwards;
}

.toast.exiting {
  animation: fadeOut 300ms ease forwards;
}
```

```jsx
import { useState } from 'react';

export default function Toast({ message, onDismiss }) {
  const [exiting, setExiting] = useState(false);

  function handleDismiss() {
    setExiting(true); // start exit animation
  }

  function handleAnimationEnd() {
    if (exiting) onDismiss(); // unmount only after animation finishes
  }

  return (
    <div
      className={`toast ${exiting ? 'exiting' : ''}`}
      onAnimationEnd={handleAnimationEnd}
    >
      <p>{message}</p>
      <button onClick={handleDismiss}>✕</button>
    </div>
  );
}
```

**Why it's better than `setTimeout`:** No magic numbers. The component unmounts exactly when the animation ends, regardless of duration.

**Best for:** Toasts, notifications, dismissible banners.

---

## 6. Staggered List Animations with `useEffect` + Index

Animate a list of items in one by one using CSS animation delay driven by each item's index.

```css
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(16px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.list-item {
  opacity: 0;
  animation: fadeUp 400ms ease forwards;
}
```

```jsx
export default function AnimatedList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li
          key={item.id}
          className="list-item"
          style={{ animationDelay: `${index * 80}ms` }}  // stagger by 80ms
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

For lists that load async, trigger the animation after data arrives:

```jsx
import { useState, useEffect } from 'react';

export default function AnimatedResults({ query }) {
  const [items, setItems] = useState([]);
  const [key, setKey] = useState(0); // increment to re-trigger animation

  useEffect(() => {
    fetch(`/api/search?q=${query}`)
      .then((res) => res.json())
      .then((data) => {
        setItems(data);
        setKey((k) => k + 1); // new key = React remounts the list = animation replays
      });
  }, [query]);

  return (
    <ul key={key}>
      {items.map((item, index) => (
        <li
          key={item.id}
          className="list-item"
          style={{ animationDelay: `${index * 60}ms` }}
        >
          {item.title}
        </li>
      ))}
    </ul>
  );
}
```

**Best for:** Card grids, search results, feature lists, onboarding screens.

---

## 7. Scroll-Triggered Animation with `useEffect` + `IntersectionObserver`

Animate elements as they scroll into view.

```css
.reveal {
  opacity: 0;
  transform: translateY(40px);
  transition: opacity 600ms ease, transform 600ms ease;
}

.reveal.in-view {
  opacity: 1;
  transform: translateY(0);
}
```

```jsx
import { useEffect, useRef } from 'react';

export default function RevealOnScroll({ children, delay = 0 }) {
  const ref = useRef(null);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          el.classList.add('in-view');
          observer.unobserve(el); // animate once, then stop watching
        }
      },
      { threshold: 0.15 } // trigger when 15% of element is visible
    );

    observer.observe(el);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={ref} className="reveal" style={{ transitionDelay: `${delay}ms` }}>
      {children}
    </div>
  );
}
```

```jsx
// Usage — wrap any content to animate it in on scroll
export default function LandingPage() {
  return (
    <main>
      <RevealOnScroll><Hero /></RevealOnScroll>
      <RevealOnScroll delay={100}><Features /></RevealOnScroll>
      <RevealOnScroll delay={200}><Pricing /></RevealOnScroll>
    </main>
  );
}
```

**Best for:** Landing pages, long-form content, marketing sections.

---

## 8. CSS Variables + State — Dynamic Animations

Drive CSS animations with dynamic values by setting CSS custom properties via inline styles.

```css
.progress-bar {
  width: 0%;
  height: 8px;
  background: #3b82f6;
  transition: width 600ms cubic-bezier(0.4, 0, 0.2, 1);
}

.spinner {
  animation: spin var(--speed) linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

```jsx
import { useState } from 'react';

export default function ProgressBar({ value }) {
  return (
    <div className="progress-track">
      <div
        className="progress-bar"
        style={{ width: `${value}%` }}   // CSS transition handles the smooth animation
        role="progressbar"
        aria-valuenow={value}
      />
    </div>
  );
}

export function DynamicSpinner({ speed = 1000 }) {
  return (
    <div
      className="spinner"
      style={{ '--speed': `${speed}ms` }}  // CSS variable from React state
    />
  );
}
```

**Best for:** Progress bars, loading indicators, data visualizations where values change over time.

---

## 9. `useRef` — Imperative Animation Control

Sometimes you need direct DOM access to start/stop/reset animations imperatively.

```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%       { transform: translateX(-8px); }
  40%       { transform: translateX(8px); }
  60%       { transform: translateX(-6px); }
  80%       { transform: translateX(6px); }
}

.input-field.shaking {
  animation: shake 400ms ease;
  border-color: red;
}
```

```jsx
import { useRef } from 'react';

export default function ValidatedInput({ onSubmit }) {
  const inputRef = useRef(null);

  function handleSubmit() {
    const value = inputRef.current.value;

    if (!value.trim()) {
      const el = inputRef.current;
      el.classList.remove('shaking');         // reset so animation can re-trigger
      void el.offsetWidth;                    // force reflow — tricks browser into restarting
      el.classList.add('shaking');
      el.addEventListener(
        'animationend',
        () => el.classList.remove('shaking'),
        { once: true }
      );
      return;
    }

    onSubmit(value);
  }

  return (
    <div>
      <input ref={inputRef} className="input-field" placeholder="Type something..." />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

**`void el.offsetWidth`** forces the browser to reflow the element, resetting the animation so it can replay even if it just finished.

**Best for:** Error shake animations, attention-grabbing effects that replay on user action.

---

## 10. Putting It All Together — Animated Notification System

A complete example combining mount animation, exit animation, staggering, and auto-dismiss.

```css
@keyframes slideIn {
  from { opacity: 0; transform: translateX(100%); }
  to   { opacity: 1; transform: translateX(0); }
}

@keyframes slideOut {
  from { opacity: 1; transform: translateX(0); }
  to   { opacity: 0; transform: translateX(100%); }
}

.toast {
  animation: slideIn 300ms ease forwards;
}

.toast.exiting {
  animation: slideOut 300ms ease forwards;
}
```

```jsx
import { useState, useEffect } from 'react';

function Toast({ id, message, type = 'info', onRemove }) {
  const [exiting, setExiting] = useState(false);

  // Auto-dismiss after 4 seconds
  useEffect(() => {
    const timer = setTimeout(() => setExiting(true), 4000);
    return () => clearTimeout(timer);
  }, []);

  return (
    <div
      className={`toast toast-${type} ${exiting ? 'exiting' : ''}`}
      onAnimationEnd={() => exiting && onRemove(id)}
    >
      <span>{message}</span>
      <button onClick={() => setExiting(true)}>✕</button>
    </div>
  );
}

export default function NotificationSystem() {
  const [toasts, setToasts] = useState([]);

  function addToast(message, type) {
    setToasts((prev) => [...prev, { id: Date.now(), message, type }]);
  }

  function removeToast(id) {
    setToasts((prev) => prev.filter((t) => t.id !== id));
  }

  return (
    <div>
      <button onClick={() => addToast('File saved!', 'success')}>Success Toast</button>
      <button onClick={() => addToast('Something went wrong.', 'error')}>Error Toast</button>

      <div className="toast-container">
        {toasts.map((toast, index) => (
          <Toast
            key={toast.id}
            {...toast}
            onRemove={removeToast}
            style={{ animationDelay: `${index * 50}ms` }}
          />
        ))}
      </div>
    </div>
  );
}
```

---

## Pattern Comparison

| Pattern                          | Enter Anim | Exit Anim | Needs `useEffect`? | Best For                          |
|----------------------------------|------------|-----------|-------------------|-----------------------------------|
| CSS class toggle                 | ✅         | ✅        | Optional          | Panels, tabs, toggles             |
| CSS keyframe on mount            | ✅         | ❌        | No                | Alerts, badges, icons             |
| Delayed class via `useEffect`    | ✅         | ❌        | Yes               | Drawers, sidebars, slides         |
| `setTimeout` unmount delay       | ✅         | ✅        | Yes               | Modals, overlays                  |
| `onAnimationEnd` unmount         | ✅         | ✅        | No                | Toasts, dismissible elements      |
| Staggered index delay            | ✅         | ❌        | Optional          | Lists, grids, cards               |
| `IntersectionObserver`           | ✅         | ❌        | Yes               | Scroll-triggered reveals          |
| CSS variables from state         | ✅         | ✅        | No                | Progress bars, dynamic values     |
| `useRef` imperative              | ✅         | ✅        | No                | Replay animations, error shakes   |