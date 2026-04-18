# JavaScript — Interview Prep

## Table of Contents
- [1. Types & Type Coercion](#1-types--type-coercion)
- [2. Scope, Closures & Hoisting](#2-scope-closures--hoisting)
- [3. `this` Keyword](#3-this-keyword)
- [4. Prototypes & Inheritance](#4-prototypes--inheritance)
- [5. Async JavaScript](#5-async-javascript)
- [6. Functions & Execution](#6-functions--execution)
- [7. Objects & Arrays](#7-objects--arrays)
- [8. ES6+ Features](#8-es6-features)
- [9. Modules](#9-modules)
- [10. Common Gotchas & Thought-Provoking Questions](#10-common-gotchas--thought-provoking-questions)

---

## 1. Types & Type Coercion

**Q: What are JavaScript's data types?**

**Primitives** (immutable, stored by value):
- `number`, `string`, `boolean`, `null`, `undefined`, `symbol`, `bigint`

**Reference types** (mutable, stored by reference):
- `object` (includes arrays, functions, dates, etc.)

```js
typeof 42          // "number"
typeof "hello"     // "string"
typeof null        // "object"  ← famous bug, null is NOT an object
typeof undefined   // "undefined"
typeof []          // "object"  ← arrays are objects
typeof function(){} // "function"
```

---

**Q: What is the difference between `==` and `===`?**

- `===` (strict equality) — compares **value AND type**. No coercion.
- `==` (loose equality) — compares after **type coercion**. JavaScript converts one or both operands to a common type first.

```js
0 == false      // true  — false coerces to 0
0 === false     // false — different types

"" == false     // true
null == undefined  // true  — special rule
null === undefined // false

[] == false     // true  — [] → "" → 0, false → 0
[] === false    // false
```

Always use `===`. The coercion rules in `==` are a source of subtle bugs and nearly no one memorizes them correctly.

---

**Q: What is `null` vs `undefined`?**

- `undefined` — a variable has been declared but not assigned a value. Also the return value of functions that don't return anything. Also what you get when accessing a non-existent object property.
- `null` — an intentional absence of a value. You set it explicitly.

```js
let x;           // x is undefined
let y = null;    // y is explicitly "nothing"

function greet() {}
greet()          // returns undefined
```

Both are falsy. `null == undefined` is `true` but `null === undefined` is `false`.

---

**Q: What is type coercion and when does it surprise you?**

JavaScript automatically converts types in certain contexts — this is coercion.

```js
"5" + 3        // "53"  — number coerces to string (+ prefers string)
"5" - 3        // 2     — string coerces to number (- is numeric only)
"5" * "3"      // 15    — both coerce to numbers

true + true    // 2
false + 1      // 1
"" + 0         // "0"

if ("0") { }   // runs — non-empty string is truthy
if (0) { }     // skipped — 0 is falsy
```

**Falsy values:** `false`, `0`, `""`, `null`, `undefined`, `NaN`, `0n`  
Everything else is truthy, including `"0"`, `[]`, `{}`.

---

## 2. Scope, Closures & Hoisting

**Q: What is the difference between `var`, `let`, and `const`?**

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisted | Yes (initialized as `undefined`) | Yes (but TDZ — not accessible before declaration) | Yes (but TDZ) |
| Re-declarable | Yes | No | No |
| Re-assignable | Yes | Yes | No |

```js
function example() {
    if (true) {
        var x = 1;   // function-scoped — visible outside the if
        let y = 2;   // block-scoped — not visible outside
    }
    console.log(x);  // 1
    console.log(y);  // ReferenceError
}
```

Never use `var` in modern code. Default to `const`; use `let` when you need to reassign.

---

**Q: What is hoisting?**

Hoisting is JavaScript's behavior of moving declarations to the top of their scope during the compilation phase — before code runs.

- **`var` declarations** are hoisted and initialized as `undefined`.
- **Function declarations** are fully hoisted (definition and body).
- **`let`/`const`** are hoisted but placed in a **Temporal Dead Zone (TDZ)** — accessing them before declaration throws `ReferenceError`.

```js
console.log(x);    // undefined — var hoisted, not the value
var x = 5;

greet();           // works — function declaration fully hoisted
function greet() { console.log("hi"); }

console.log(y);    // ReferenceError — TDZ
let y = 10;
```

---

**Q: What is a closure?**

A closure is a function that **remembers the variables from its outer scope even after the outer function has finished executing**. The inner function "closes over" the outer scope.

```js
function makeCounter() {
    let count = 0;
    return function() {
        count++;          // count is still accessible
        return count;
    };
}

const counter = makeCounter();
counter();  // 1
counter();  // 2
counter();  // 3
```

`count` lives in `makeCounter`'s scope, which was destroyed after the call — but the returned function still has a reference to it. This is a closure.

**Real-world use cases:** data privacy, factory functions, memoization, event handlers that remember state.

**Classic closure gotcha:**
```js
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 0);  // prints 3, 3, 3 — not 0, 1, 2
}
// Fix: use let (block-scoped, new binding per iteration)
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 0);  // prints 0, 1, 2
}
```

---

**Q: What is the scope chain?**

When a variable is referenced, JavaScript looks in the current scope first, then walks up through enclosing scopes until it reaches the global scope. This chain of scopes is the **scope chain**.

```js
const global = "global";

function outer() {
    const outerVar = "outer";
    function inner() {
        const innerVar = "inner";
        console.log(innerVar);  // found in inner scope
        console.log(outerVar);  // found in outer scope
        console.log(global);    // found in global scope
    }
    inner();
}
```

---

## 3. `this` Keyword

**Q: How does `this` work in JavaScript?**

`this` refers to the object that the function was **called on** — not where the function was defined. Its value depends entirely on the call site.

```js
// 1. Global context
console.log(this);   // window (browser) / global (Node) / undefined (strict mode)

// 2. Method call — this = the object before the dot
const user = {
    name: "Alice",
    greet() { console.log(this.name); }
};
user.greet();   // "Alice"

// 3. Losing context — common bug
const fn = user.greet;
fn();           // undefined — this is now global/undefined

// 4. Explicit binding
fn.call(user);          // "Alice"
fn.apply(user);         // "Alice"
const bound = fn.bind(user);
bound();                // "Alice"

// 5. Arrow functions — inherit this from enclosing lexical scope, never own this
const obj = {
    name: "Bob",
    greet: () => console.log(this.name)  // this is NOT obj — it's the outer scope
};
obj.greet();   // undefined

// 6. new — this = the newly created object
function Person(name) { this.name = name; }
const p = new Person("Carol");
p.name;  // "Carol"
```

---

**Q: Arrow function vs regular function — what's the key difference beyond syntax?**

Arrow functions differ in two important ways:
1. **No own `this`** — they inherit `this` from the surrounding lexical scope. Fixes the "lost `this`" problem in callbacks.
2. **No `arguments` object** — use rest parameters instead.
3. Cannot be used as constructors (`new ArrowFn()` throws).
4. No `prototype` property.

```js
class Timer {
    constructor() { this.ticks = 0; }

    start() {
        // Regular function: this would be undefined inside the callback
        setInterval(() => {  // Arrow function: this = Timer instance ✓
            this.ticks++;
        }, 1000);
    }
}
```

---

## 4. Prototypes & Inheritance

**Q: How does prototypal inheritance work?**

Every JavaScript object has an internal `[[Prototype]]` link to another object (its prototype). When you access a property, JavaScript looks on the object first, then walks up the **prototype chain** until it finds it or hits `null`.

```js
const animal = {
    speak() { console.log("..."); }
};

const dog = Object.create(animal);  // dog's prototype = animal
dog.name = "Rex";

dog.speak();  // found on animal via prototype chain
dog.name;     // found directly on dog

Object.getPrototypeOf(dog) === animal;  // true
```

---

**Q: What does the `class` keyword actually do? Is it different from prototypes?**

`class` is **syntactic sugar** over JavaScript's existing prototype-based system. Under the hood, nothing changes — it's still prototypal inheritance.

```js
class Animal {
    constructor(name) { this.name = name; }
    speak() { console.log(`${this.name} speaks`); }
}

class Dog extends Animal {
    speak() { console.log(`${this.name} barks`); }
}

const d = new Dog("Rex");
d.speak();   // "Rex barks"

// Under the hood:
// Dog.prototype.__proto__ === Animal.prototype  → true
// d.__proto__ === Dog.prototype                 → true
```

`extends` sets up the prototype chain. `super()` calls the parent constructor or parent method.

---

## 5. Async JavaScript

**Q: What is the event loop?**

JavaScript is **single-threaded** — only one piece of code runs at a time. The event loop enables asynchronous behavior without blocking.

**Key components:**
- **Call stack** — where currently executing functions live. LIFO.
- **Web APIs** — browser/Node handles async ops (`setTimeout`, `fetch`, DOM events) outside the stack.
- **Task queue (macrotask queue)** — completed async callbacks wait here (`setTimeout`, `setInterval`, I/O).
- **Microtask queue** — higher priority queue for promise callbacks (`.then`, `catch`, `finally`, `await`), processed after every task before the next task.

**Execution order:**
1. Run current synchronous code (empty the call stack).
2. Process ALL microtasks (promise callbacks).
3. Render (browser).
4. Pick one macrotask, run it.
5. Repeat.

```js
console.log("1");
setTimeout(() => console.log("2"), 0);   // macrotask
Promise.resolve().then(() => console.log("3"));  // microtask
console.log("4");

// Output: 1, 4, 3, 2
// Sync first → microtask → macrotask
```

---

**Q: Callbacks vs Promises vs async/await — what's the difference?**

All three are ways to handle async operations — each is built on the previous.

**Callbacks** — pass a function to be called when async work finishes. Simple, but leads to "callback hell" when chained.
```js
readFile("a.txt", (err, data) => {
    readFile("b.txt", (err, data2) => {
        // pyramid of doom
    });
});
```

**Promises** — represent a future value. Chainable with `.then()` / `.catch()`. Avoids nesting.
```js
fetch("/api/user")
    .then(res => res.json())
    .then(user => console.log(user))
    .catch(err => console.error(err));
```

**async/await** — syntactic sugar over promises. Write async code that reads like synchronous code.
```js
async function loadUser() {
    try {
        const res = await fetch("/api/user");
        const user = await res.json();
        console.log(user);
    } catch (err) {
        console.error(err);
    }
}
```

`await` pauses execution of the `async` function until the promise resolves. It does NOT block the event loop — other code can still run.

---

**Q: What is `Promise.all` vs `Promise.allSettled` vs `Promise.race`?**

```js
const p1 = fetch("/api/users");
const p2 = fetch("/api/products");
const p3 = fetch("/api/orders");

// Promise.all — resolves when ALL resolve; rejects immediately if ANY rejects
const [users, products, orders] = await Promise.all([p1, p2, p3]);

// Promise.allSettled — waits for ALL to finish regardless of success/failure
const results = await Promise.allSettled([p1, p2, p3]);
results.forEach(r => {
    if (r.status === "fulfilled") console.log(r.value);
    else console.error(r.reason);
});

// Promise.race — resolves/rejects with the FIRST settled promise
const fastest = await Promise.race([p1, p2, p3]);
```

Use `Promise.all` when you need all results and any failure should abort. Use `Promise.allSettled` when you want to handle each outcome individually.

---

## 6. Functions & Execution

**Q: What is the difference between a function declaration and a function expression?**

```js
// Function declaration — hoisted fully, available before its line
function add(a, b) { return a + b; }

// Function expression — not hoisted, variable is in TDZ/undefined before this line
const multiply = function(a, b) { return a * b; };

// Named function expression — name only visible inside itself (useful for recursion/debugging)
const factorial = function fact(n) {
    return n <= 1 ? 1 : n * fact(n - 1);
};
```

---

**Q: What is an IIFE (Immediately Invoked Function Expression)?**

A function defined and called immediately. Historically used to create a private scope and avoid polluting global scope — less needed now with `let`/`const` and ES modules.

```js
(function() {
    const privateVar = "only here";
    console.log(privateVar);
})();

// Arrow IIFE
(() => {
    console.log("runs immediately");
})();
```

---

**Q: What is memoization?**

Memoization is caching the result of a function call so that repeated calls with the same arguments skip recomputation.

```js
function memoize(fn) {
    const cache = new Map();
    return function(...args) {
        const key = JSON.stringify(args);
        if (cache.has(key)) return cache.get(key);
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

const expensiveCalc = memoize((n) => {
    console.log("computing...");
    return n * n;
});

expensiveCalc(5);  // computing... → 25
expensiveCalc(5);  // (cached) → 25
```

---

## 7. Objects & Arrays

**Q: What is the difference between shallow copy and deep copy?**

- **Shallow copy** — copies the top-level structure. Nested objects/arrays are still shared references.
- **Deep copy** — recursively copies everything. No shared references.

```js
const original = { a: 1, b: { c: 2 } };

// Shallow copy methods
const shallow1 = { ...original };
const shallow2 = Object.assign({}, original);

shallow1.b.c = 99;
console.log(original.b.c);  // 99 — oops, shared reference

// Deep copy (simple cases)
const deep = JSON.parse(JSON.stringify(original));  // works for plain data, loses functions/dates
deep.b.c = 99;
console.log(original.b.c);  // 2 — independent
```

`structuredClone(original)` (modern browsers/Node 17+) is the proper deep clone API.

---

**Q: What are useful array methods every developer should know?**

```js
const nums = [1, 2, 3, 4, 5];

nums.map(x => x * 2)         // [2,4,6,8,10]  — transform, returns new array
nums.filter(x => x % 2 === 0) // [2,4]         — keep matching items
nums.reduce((acc, x) => acc + x, 0)  // 15     — fold into single value
nums.find(x => x > 3)        // 4              — first match or undefined
nums.findIndex(x => x > 3)   // 3              — index of first match
nums.some(x => x > 4)        // true           — at least one match
nums.every(x => x > 0)       // true           — all match
nums.flat()                  // flattens nested arrays one level
nums.flatMap(x => [x, x*2]) // map + flatten

// Mutation methods (modify original)
nums.push(6)       // add to end
nums.pop()         // remove from end
nums.unshift(0)    // add to start
nums.shift()       // remove from start
nums.splice(1, 2)  // remove/insert at index
nums.sort((a, b) => a - b)  // always provide comparator for numbers!
```

---

## 8. ES6+ Features

**Q: What are destructuring, spread, and rest?**

```js
// Destructuring — unpack values from arrays/objects
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

const { name, age, city = "Unknown" } = { name: "Alice", age: 30 };
// name="Alice", age=30, city="Unknown" (default)

// Rename while destructuring
const { name: userName } = { name: "Alice" };

// Spread — expand iterable/object into individual elements
const arr = [1, 2, 3];
const copy = [...arr, 4, 5];       // [1,2,3,4,5]
const merged = { ...obj1, ...obj2 }; // merge objects (right wins on conflict)

// Rest — collect remaining args into array
function sum(first, ...others) {
    return others.reduce((a, b) => a + b, first);
}
sum(1, 2, 3, 4);  // 10
```

---

**Q: What are template literals and tagged templates?**

```js
const name = "Alice";
const age = 30;

// Template literal — backticks, interpolation, multi-line
const msg = `Hello, ${name}! You are ${age} years old.
Second line without \n.`;

// Expressions inside ${}
const price = `Total: $${(1.23 * 100).toFixed(2)}`;

// Tagged template — function processes the template
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) =>
        result + str + (values[i] ? `<b>${values[i]}</b>` : ""), "");
}
highlight`Hello ${name}, you are ${age} years old`;
// "Hello <b>Alice</b>, you are <b>30</b> years old"
```

---

**Q: What are optional chaining (`?.`) and nullish coalescing (`??`)?**

```js
const user = { address: { city: "NYC" } };

// Optional chaining — short-circuits to undefined if any part is null/undefined
user?.address?.city       // "NYC"
user?.phone?.number       // undefined — no error thrown
user?.getName?.()         // call method only if it exists

// Nullish coalescing — returns right side only if left is null or undefined
// (NOT for other falsy values like 0 or "")
const city = user?.city ?? "Unknown";   // "Unknown"
const count = 0 ?? 42;                 // 0 — 0 is not null/undefined
const count2 = 0 || 42;               // 42 — || treats 0 as falsy

// Combine both
const zip = user?.address?.zip ?? "N/A";
```

---

## 9. Modules

**Q: What are ES modules and how do they work?**

ES modules (`import`/`export`) are the standard module system for JavaScript (ES2015+). Each file is its own module with its own scope — nothing leaks to global.

```js
// math.js — named exports
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export default function multiply(a, b) { return a * b; }  // default export

// main.js — importing
import multiply, { PI, add } from "./math.js";   // default + named
import * as math from "./math.js";               // namespace import
import { add as sum } from "./math.js";          // rename on import
```

Key properties of ES modules:
- **Static** — imports/exports are resolved at parse time, not runtime. Enables tree-shaking (dead code elimination by bundlers).
- **Live bindings** — imported names reflect the current value of the export (unlike CommonJS which copies the value at require time).
- **Strict mode** by default.
- **Async** loading in browsers — `<script type="module">`.

---

## 10. Common Gotchas & Thought-Provoking Questions

**Q: What does this output and why?**

```js
console.log(typeof NaN);     // "number" — NaN is of type number (it means "invalid number")
console.log(NaN === NaN);    // false    — NaN is the only value not equal to itself
console.log(isNaN("hello")); // true     — coerces first, then checks (misleading)
console.log(Number.isNaN("hello")); // false — strict: only true if value IS NaN

console.log(0.1 + 0.2 === 0.3); // false — floating point precision
console.log(0.1 + 0.2);         // 0.30000000000000004
// Fix: Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON
```

---

**Q: What is the difference between `Object.freeze` and `const`?**

- **`const`** — the variable binding cannot be reassigned. The object it points to can still be mutated.
- **`Object.freeze(obj)`** — makes the object's properties non-writable and non-configurable. Shallow — nested objects are not frozen.

```js
const obj = { x: 1 };
obj.x = 99;          // works — const only protects the binding, not the content
obj = {};            // TypeError — can't reassign const

const frozen = Object.freeze({ x: 1, nested: { y: 2 } });
frozen.x = 99;       // silently fails (throws in strict mode)
frozen.nested.y = 99; // works — freeze is shallow
```

---

**Q: What is event bubbling and capturing?**

When an event fires on a DOM element, it travels in two phases:
1. **Capturing** — top-down: from `document` down to the target element.
2. **Bubbling** — bottom-up: from the target back up to `document`.

By default, event listeners fire during the **bubbling** phase.

```js
document.querySelector("#parent").addEventListener("click", handler);       // bubbling (default)
document.querySelector("#parent").addEventListener("click", handler, true); // capturing

// Stop propagation — prevent the event from traveling further
element.addEventListener("click", (e) => {
    e.stopPropagation();  // won't reach parent listeners
});

// Event delegation — attach one listener to a parent instead of many to children
document.querySelector("#list").addEventListener("click", (e) => {
    if (e.target.matches("li")) {
        console.log("Clicked:", e.target.textContent);
    }
});
```

**Event delegation** is a performance pattern: one listener handles all child events via bubbling. Especially useful for dynamically added elements.
