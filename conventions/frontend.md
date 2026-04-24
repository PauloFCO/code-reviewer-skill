# Frontend Conventions: React/TypeScript, Clean Architecture, JS Best Practices

## React Best Practices

### Rules of Hooks
Hooks must only be called at the top level of a React function. Never inside loops, conditions, or nested functions.

**Violations:**
```tsx
// BAD — hook inside a condition
function MyComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [data, setData] = useState(null); // VIOLATION
  }
}

// BAD — hook inside a loop
function MyList({ items }) {
  return items.map(item => {
    const [selected, setSelected] = useState(false); // VIOLATION
    return <div onClick={() => setSelected(true)}>{item.name}</div>;
  });
}
```

### Avoid useEffect Abuse
`useEffect` should only be used for true side effects (subscriptions, manual DOM manipulation, external system sync). Avoid using it for data transformations or derived state.

**Violations:**
```tsx
// BAD — deriving state inside useEffect
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`); // should be a useMemo or plain variable
}, [firstName, lastName]);

// BAD — fetching without cleanup / race condition handling
useEffect(() => {
  fetch('/api/data').then(r => r.json()).then(setData); // no cleanup, no abort controller
}, [id]);
```

**Good pattern:**
```tsx
// GOOD — derived value without effect
const fullName = `${firstName} ${lastName}`;

// GOOD — fetch with cleanup
useEffect(() => {
  const controller = new AbortController();
  fetch('/api/data', { signal: controller.signal })
    .then(r => r.json())
    .then(setData)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [id]);
```

### Component Composition
Prefer composition (small, focused components) over monolithic components with many props.

**Violations:**
- Components over 150–200 lines
- More than 5–6 props in a component — signals it should be split
- Passing data 3+ levels deep via props (prop drilling) — use Context or state management

**Prop drilling violation:**
```tsx
// BAD — user passed through 3 levels just to reach a leaf
<Dashboard user={user}>
  <Sidebar user={user}>
    <UserAvatar user={user} />
  </Sidebar>
</Dashboard>
```

### Keys in Lists
Always provide stable, unique keys when rendering lists. Never use array index as key when the list can reorder or be filtered.

**Violations:**
```tsx
// BAD — index as key
items.map((item, index) => <Item key={index} {...item} />)

// BAD — no key
items.map(item => <Item {...item} />)
```

### Memoization
Use `React.memo`, `useMemo`, and `useCallback` only when there is a measured performance problem. Don't over-memoize.

**Violations (over-memoization):**
```tsx
// BAD — useMemo on a trivial computation
const doubled = useMemo(() => value * 2, [value]);

// BAD — useCallback on a function that doesn't flow to a memo'd child
const handleClick = useCallback(() => setOpen(true), []);
```

**Violations (missing memoization in hot paths):**
- A component that re-renders frequently passing a new object literal `{}` or array `[]` as a prop to a `React.memo` child (defeats memoization)

### Controlled vs Uncontrolled Components
Be explicit about whether a form input is controlled (value + onChange) or uncontrolled (ref). Don't mix them.

**Violation:**
```tsx
// BAD — switching between controlled and uncontrolled
<input value={value || ''} onChange={...} /> // falsy value makes it uncontrolled initially
```

---

## TypeScript Best Practices

### Avoid `any`
`any` disables type checking. Use `unknown`, proper generics, or explicit types instead.

**Violations:**
```ts
// BAD
function process(data: any) { ... }
const result: any = await fetchData();

// GOOD
function process(data: unknown) {
  if (typeof data === 'string') { ... }
}
```

### Explicit Return Types
Public functions and methods should have explicit return types for better readability and catch regressions.

**Violations:**
```ts
// BAD — implicit return type on exported function
export function getUser(id: string) { // what does this return?
  return userRepository.findById(id);
}
```

### Interface vs Type
Use `interface` for object shapes that may be extended; use `type` for unions, intersections, and aliases. Don't mix styles inconsistently.

### No Type Assertions Without Guards
Avoid `as SomeType` without validating the value first. Prefer type guards.

**Violation:**
```ts
// BAD — forced cast without validation
const user = response.data as User;

// GOOD — validate first
function isUser(value: unknown): value is User {
  return typeof value === 'object' && value !== null && 'id' in value;
}
```

### Generics
Use generics to create reusable, type-safe utilities instead of repeating code for different types.

---

## Clean Architecture — Frontend

The frontend should also respect layered architecture with zero coupling between layers.

```
Presentation Layer    ← React components, pages, styles. No business logic.
Application Layer     ← Use cases / services. Orchestrates domain logic. No UI.
Domain Layer          ← Entities, value objects, business rules. Pure TypeScript, no React.
Infrastructure Layer  ← API clients, localStorage, analytics. Implements interfaces.
```

### Layer Violations to Detect

**UI component containing business logic:**
```tsx
// BAD — component doing domain logic
function OrderSummary({ order }) {
  const discount = order.items.length > 5 ? order.total * 0.1 : 0; // business rule in UI
  const tax = order.total * 0.21; // business rule in UI
  return <div>{order.total - discount + tax}</div>;
}
```

**Application layer importing UI framework:**
```ts
// BAD — use case importing React
import { useState } from 'react'; // application layer must not depend on React
export class CreateOrderUseCase { ... }
```

**Direct API calls from components:**
```tsx
// BAD — component calling API directly
function UserProfile({ id }) {
  useEffect(() => {
    fetch(`/api/users/${id}`).then(...); // should go through a service/repository
  }, [id]);
}
```

**Good pattern — dependency injection via props/context:**
```tsx
// GOOD — component receives service via props or context
function UserProfile({ id, userService }: { id: string; userService: UserService }) {
  const [user, setUser] = useState<User | null>(null);
  useEffect(() => {
    userService.getById(id).then(setUser);
  }, [id, userService]);
}
```

### Zero Coupling Between Layers
- Domain layer must have zero imports from infrastructure, application, or presentation
- Application layer imports from domain only (via interfaces/ports)
- Infrastructure layer imports from application/domain interfaces only — never implements business logic
- Presentation layer imports from application layer (services/use cases) — never directly from infrastructure

---

## JavaScript Best Practices

### Variable Declarations
Always use `const` or `let`. Never `var`.

**Violation:** `var count = 0;`

### Async/Await
Use `async/await` over raw Promises with `.then()/.catch()` chains for readability. Always handle errors.

**Violations:**
```js
// BAD — unhandled rejection
async function loadData() {
  const data = await fetchData(); // no try/catch
  return data;
}

// BAD — mixing await and .then()
async function mixed() {
  const result = await fetch(url).then(r => r.json()); // confusing
}
```

**Good:**
```js
async function loadData() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    logger.error('Failed to load data', error);
    throw error;
  }
}
```

### Destructuring
Use destructuring for objects and arrays when it improves clarity. Don't over-destructure nested structures in one line.

```js
// GOOD
const { name, email } = user;
const [first, ...rest] = items;

// BAD — overly nested destructuring
const { a: { b: { c } } } = obj; // hard to read, fragile
```

### No `console.log` in Production Code
Remove all `console.log`, `console.debug`, and `console.warn` statements that were used for debugging.

**Violations:** Any `console.log(...)` in non-test, non-utility code.

### Null Coalescing and Optional Chaining
Use `?.` and `??` instead of verbose null checks.

```js
// BAD
const name = user && user.profile && user.profile.name ? user.profile.name : 'Unknown';

// GOOD
const name = user?.profile?.name ?? 'Unknown';
```

### Spread and Rest Operators
Use spread for shallow copies and merging, rest for collecting arguments. Avoid mutating objects/arrays directly.

**Violations:**
```js
// BAD — direct mutation
state.items.push(newItem); // mutates original array
Object.assign(state, updates); // mutates original object

// GOOD
const newItems = [...state.items, newItem];
const newState = { ...state, ...updates };
```

### Explicit Error Handling
Never silently ignore errors. Every `try/catch` must either handle the error meaningfully, log it, or re-throw it.

**Violation:**
```js
// BAD — swallowed error
try {
  await riskyOperation();
} catch (e) {
  // nothing here
}
```
