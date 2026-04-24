# Default Conventions: SOLID, OOP, Clean Code, Hexagonal Architecture

## SOLID Principles

### S — Single Responsibility Principle (SRP)
A class or module should have only one reason to change. Each class/function should do one thing only.

**Violations to detect:**
- A class that handles business logic AND persistence AND HTTP responses
- A function that validates input, transforms data, AND sends an email
- Methods with names like `processAndSaveAndNotify()`

**Java example of violation:**
```java
// BAD: UserService does too many things
public class UserService {
    public void registerUser(User user) {
        validate(user);           // validation
        userRepository.save(user); // persistence
        emailService.send(user);   // notification
        auditLog.write(user);      // logging
    }
}
```

**JavaScript/TypeScript violation:**
```ts
// BAD
function handleUser(user) {
  if (!user.email) throw new Error('invalid');
  db.save(user);
  sendWelcomeEmail(user);
  analytics.track('user_created', user);
}
```

---

### O — Open/Closed Principle (OCP)
Software entities should be open for extension but closed for modification. Add behavior via new code, not by editing existing code.

**Violations to detect:**
- Long `if/else` or `switch` chains checking a type/role to decide behavior (should use polymorphism)
- Adding a new case to a switch requires modifying existing classes

**Violation example:**
```java
// BAD: every new shape requires modifying this class
public double calculateArea(Shape shape) {
    if (shape.type.equals("circle")) return Math.PI * shape.radius * shape.radius;
    else if (shape.type.equals("square")) return shape.side * shape.side;
    // add new shape → must modify here
}
```

---

### L — Liskov Substitution Principle (LSP)
Subtypes must be substitutable for their base types without altering correctness.

**Violations to detect:**
- Overriding a method to throw `UnsupportedOperationException` or return `null`
- A subclass that weakens the contract of the parent (e.g., ignores parameters, always returns a default)
- A subclass that requires stricter preconditions than the parent

---

### I — Interface Segregation Principle (ISP)
Clients should not be forced to depend on interfaces they don't use. Prefer small, focused interfaces over large "fat" ones.

**Violations to detect:**
- An interface with 10+ methods where implementations leave half as `throw new UnsupportedOperationException()`
- A class implementing an interface and marking several methods as `// not needed`

---

### D — Dependency Inversion Principle (DIP)
Depend on abstractions, not concretions. High-level modules should not depend on low-level modules.

**Violations to detect:**
- `new MySQLRepository()` directly inside a service class (should be injected)
- Importing infrastructure classes (e.g., `import { MySQLConnection }`) from domain/application layers
- No constructor injection — instead, instantiating dependencies inline

---

## Object-Oriented Programming (OOP) Principles

### Encapsulation
Hide internal state. Expose only what's necessary via public API.

**Violations:**
- Public fields instead of private fields + getters/setters (Java)
- Directly mutating object properties from outside the object
- Exposing internal implementation details through the public API

```java
// BAD
public class BankAccount {
    public double balance; // should be private
}
```

### Abstraction
Model concepts at the right level. Don't expose how something works, expose what it does.

**Violations:**
- Method names that reveal implementation details (e.g., `fetchFromMySQLAndDeserialize()` instead of `getUser()`)
- Leaking internal data structures through the public API

### Composition over Inheritance
Prefer composing objects over deep inheritance hierarchies.

**Violations:**
- Inheritance chains deeper than 2-3 levels for the sake of code reuse
- Using inheritance where the "is-a" relationship doesn't truly hold
- Inheriting just to reuse a utility method (use composition or a static helper instead)

### Polymorphism
Use polymorphism to replace conditional logic based on type.

**Violations:**
- `instanceof` checks to determine behavior (should be polymorphic dispatch)
- Type fields + switch/if chains where polymorphism would clean it up

---

## Clean Code

### Meaningful Names
Names should reveal intent. Avoid abbreviations, single-letter variables (except loop counters), and misleading names.

**Violations:**
- Variables named `d`, `tmp`, `data`, `info`, `flag`, `obj`
- Boolean variables not phrased as questions: `status` instead of `isActive`
- Functions named `process()`, `handle()`, `manage()` with no specifics
- Inconsistent naming (camelCase mixed with snake_case in the same language)

### Small Functions
Functions should do one thing. If a function is >20 lines or has multiple levels of abstraction, it likely does too much.

**Violations:**
- Functions longer than 20–25 lines
- Functions with deeply nested conditionals (more than 2–3 levels)
- Functions that could be split into "prepare data" + "execute" + "report result"

### DRY — Don't Repeat Yourself
Avoid duplicating logic. Extract repeated code into a shared function or constant.

**Violations:**
- Identical or near-identical code blocks in multiple places
- Magic numbers or strings repeated across the codebase (use named constants)

### No Magic Numbers or Strings
Use named constants instead of raw values embedded in logic.

```java
// BAD
if (user.age >= 18) { ... }

// GOOD
private static final int MINIMUM_LEGAL_AGE = 18;
if (user.age >= MINIMUM_LEGAL_AGE) { ... }
```

### Comments
Comments should explain WHY, not WHAT. The code should be self-explanatory.

**Violations:**
- Comments that restate what the code does: `// increment counter` above `count++`
- Commented-out dead code left behind
- TODO comments that are stale or have no owner
- Multi-paragraph docstrings describing every parameter when names are self-evident

### No Dead Code
Remove unused variables, imports, methods, and commented-out blocks.

**Violations:**
- `import` statements for unused classes/modules
- Private methods that are never called
- Variables declared but never read
- Large blocks of commented-out code

### Error Handling
Errors should be handled explicitly. Don't swallow exceptions or use generic catch-all handlers without re-throwing.

**Violations:**
- Empty `catch` blocks
- `catch (Exception e) { }` with no logging or re-throw
- Returning `null` to signal an error instead of throwing or using Optional/Result types
- Ignoring Promise rejections in JavaScript (no `.catch()` or `try/catch` around `await`)

---

## Hexagonal Architecture (Ports & Adapters)

The domain is the center. Ports define what the domain needs. Adapters implement those ports.

**Layer rules:**
- **Domain layer**: pure business logic only. No framework imports, no HTTP, no SQL, no file I/O.
- **Application layer (use cases)**: orchestrates domain objects. May depend on port interfaces but NOT on adapters.
- **Infrastructure layer (adapters)**: implements ports. May import frameworks, databases, HTTP clients.

### Critical violations:

**Domain importing infrastructure:**
```java
// BAD — domain class importing a JPA/SQL dependency
import javax.persistence.Entity; // infrastructure concern in domain
import org.springframework.data.jpa.repository.JpaRepository; // in domain layer
```

**Application layer directly instantiating infrastructure:**
```java
// BAD — use case calling a concrete repository, not an interface
public class CreateOrderUseCase {
    private MySQLOrderRepository repository = new MySQLOrderRepository(); // DIP violation + hexagonal violation
}
```

**Use case depending on HTTP request/response objects:**
```java
// BAD
public class GetUserUseCase {
    public ResponseEntity<User> execute(HttpServletRequest request) { ... }
}
```

**Correct structure:**
```
domain/
  model/         ← entities, value objects — no external dependencies
  port/
    in/          ← input ports (use case interfaces)
    out/         ← output ports (repository interfaces, external service interfaces)

application/
  usecase/       ← implements input ports, depends only on output port interfaces

infrastructure/
  persistence/   ← implements output ports (e.g., JPA repositories)
  web/           ← HTTP controllers (Spring, Express) — calls use cases via input ports
  external/      ← third-party service adapters
```

**Violations to detect:**
- Domain model annotated with framework annotations (`@Entity`, `@Column`, `@JsonProperty` in Java)
- Use case importing from `infrastructure` package
- Controller containing business logic beyond routing/mapping
- Repository interface defined in `infrastructure` instead of `domain/port/out`
