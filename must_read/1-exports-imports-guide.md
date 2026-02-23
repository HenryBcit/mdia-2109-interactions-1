# Exports & Imports in JavaScript

JavaScript has two module systems: **CommonJS** (used in Node.js) and **ES Modules** (modern standard). Here's how both work.

---

## ES Modules (ESM)

### Named Exports

Named exports allow you to export **multiple values** from a single file. Each export must be imported using its exact name.

```js
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
```

```js
// main.js
import { PI, add, subtract } from './math.js';

console.log(PI);        // 3.14159
console.log(add(2, 3)); // 5
```

You can also **rename** on import:

```js
import { add as sum } from './math.js';
console.log(sum(2, 3)); // 5
```

Or import **everything** as a namespace object:

```js
import * as Math from './math.js';
console.log(Math.PI);   // 3.14159
```

---

### Default Export

A file can have **only one** default export. It represents the "main" thing a module exports and can be imported with **any name** you choose.

```js
// greet.js
export default function greet(name) {
  return `Hello, ${name}!`;
}
```

```js
// main.js
import greet from './greet.js';       // ✅ any name works
import sayHello from './greet.js';    // ✅ also valid

console.log(greet('Alice'));   // Hello, Alice!
```

> **Key rule:** Default exports use no curly braces `{}`. Named exports require curly braces.

---

### Mixing Default and Named Exports

You can combine both in a single file:

```js
// utils.js
export const version = '1.0.0';          // named
export default function main() { ... }   // default
```

```js
import main, { version } from './utils.js';
```

---

## CommonJS (CJS)

Used in Node.js with `.js` or `.cjs` files (before ES Modules were widely supported).

### `module.exports`

```js
// math.js
function add(a, b) { return a + b; }
const PI = 3.14;

module.exports = { add, PI };   // export an object
```

```js
// main.js
const { add, PI } = require('./math');
```

You can also export a single value (equivalent of default export):

```js
// greet.js
module.exports = function greet(name) {
  return `Hello, ${name}!`;
};
```

```js
// main.js
const greet = require('./greet');
console.log(greet('Alice'));
```

### `exports` shorthand

`exports` is a reference to `module.exports`. You can use it to attach named properties:

```js
// math.js
exports.add = (a, b) => a + b;
exports.PI = 3.14;
```

> ⚠️ **Don't reassign `exports` directly** (e.g. `exports = something`). This breaks the reference to `module.exports`. Always use `module.exports =` for a full replacement.

---

## Quick Comparison

| Feature             | ES Modules (`import`/`export`) | CommonJS (`require`) |
|---------------------|-------------------------------|----------------------|
| Syntax              | `import` / `export`           | `require` / `module.exports` |
| Default export      | `export default`              | `module.exports = value` |
| Named exports       | `export const x = ...`        | `exports.x = ...`    |
| Loading             | Static (compile-time)         | Dynamic (runtime)    |
| Browser support     | ✅ Native                     | ❌ Needs bundler      |
| Node.js support     | ✅ (`.mjs` or `"type":"module"`) | ✅ Default           |
