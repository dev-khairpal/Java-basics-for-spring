# 📘 Module 3: Essential Modern Java for Spring Boot

> **Goal:** Master the specific modern Java features (`Collections`, `Generics`, `Optional`, `Unchecked Exceptions`, `Lambdas/Streams`, and `Annotations`) that you will write daily in Spring Boot projects.

---

## 1. Collections & Generics

### 📖 Definitions
* **Collections Framework:** A standardized set of data structure classes to store, manipulate, and query groups of objects.
* **Generics (`<T>`):** A compile-time type-safety feature that allows classes and interfaces to work with specific object types without casting.

### 💡 Real-World Analogy: Labeled Storage Boxes
* **Raw Collection (without Generics):** A plain cardboard box where anyone can throw anything (books, apples, shoes). You never know what you'll pull out until runtime.
* **Generic Collection (`List<User>`):** A box clearly labeled *"USERS ONLY"*. The compiler refuses to let you put an apple inside, eliminating `ClassCastException` bugs.

### Core Collections in Spring:
| Collection | Behavior | Spring Boot Use Case |
| :--- | :--- | :--- |
| **`List<T>`** (`ArrayList`) | Ordered, allows duplicates | Returning a list of database records (`userRepository.findAll()`) |
| **`Set<T>`** (`HashSet`) | Unordered, only unique elements | User roles (`Set<Role> roles = Set.of(Role.ADMIN, Role.USER);`) |
| **`Map<K, V>`** (`HashMap`) | Key-Value pairs | Custom API response payloads (`Map<String, Object> response;`) |

```java
// Spring Data JPA uses Generics:
// T = UserEntity, ID = Long (Primary Key)
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring automatically provides type-safe methods:
    // List<User> findAll();
    // Optional<User> findById(Long id);
    // User save(User user);
}
```

---

## 2. `Optional<T>`: Eliminating NullPointerExceptions

### 📖 Definition
`Optional<T>` is a container object which may or may not contain a non-null value. It forces developers to explicitly handle the "value absent" scenario without risking `NullPointerException`.

```java
// Spring Data JPA returns Optional when searching by ID:
Optional<User> userOptional = userRepository.findById(101L);

// Modern Spring Boot pattern using orElseThrow():
User user = userOptional.orElseThrow(() -> 
    new ResourceNotFoundException("User with ID 101 not found!")
);
```

---

## 3. Exceptions: Checked vs. Unchecked

### 📖 Definitions
* **Checked Exceptions (`extends Exception`):** Exceptions checked at compile-time. Java forces you to either catch them with `try-catch` or declare them with `throws` (e.g., `SQLException`, `IOException`).
* **Unchecked Exceptions (`extends RuntimeException`):** Exceptions that occur during program execution. The compiler does not force explicit handling (e.g., `NullPointerException`, `IllegalArgumentException`).

### 🧠 Modern Spring Architecture Rule:
Spring Framework converts all low-level checked exceptions (like SQL errors) into **Unchecked Runtime Exceptions** (`DataAccessException`). In your own business logic, **always create custom unchecked exceptions**:

```java
// 1. Custom Unchecked Business Exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}

// 2. In your Service Layer (Clean, no boilerplate try-catch needed)
public User getUserById(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
}
```

In Spring Boot, you catch this globally and convert it to HTTP 404 using `@RestControllerAdvice`:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

## 4. Lambdas & Stream API (The Spring Essentials)

### 📖 Definitions
* **Lambda Expression:** A concise, anonymous function that can be passed around as a value: `(params) -> expression`.
* **Stream API:** A functional pipeline to transform, filter, and aggregate collections without writing messy `for` loops.

### 💡 Stream Pipeline Example:
```java
List<User> users = List.of(
    new User(1L, "Alice", 25, "ADMIN"),
    new User(2L, "Bob", 17, "USER"),
    new User(3L, "Charlie", 30, "USER")
);

// 1. Filter elements by condition
List<User> adults = users.stream()
    .filter(user -> user.getAge() >= 18)
    .toList();

// 2. Map / Transform (Extract names)
List<String> usernames = users.stream()
    .map(User::getUsername) // Method reference: equivalent to u -> u.getUsername()
    .toList();

// 3. Find first matching item
User firstAdmin = users.stream()
    .filter(u -> "ADMIN".equals(u.getRole()))
    .findFirst()
    .orElseThrow(() -> new RuntimeException("No admin found in system"));
```

---

## 5. Annotations & Reflection ⭐ (The Magic Behind Spring)

### 📖 Definitions
* **Annotation (`@`):** A form of syntactic metadata added to Java code. Annotations do **not** run code directly; they serve as markers/instructions for tools and frameworks.
* **Reflection:** Java's ability to inspect and instantiate classes, constructors, methods, and annotations dynamically at runtime.

### 💡 Real-World Analogy: Shipping Labels
* Think of annotations as **stickers on a parcel**: `"FRAGILE"`, `"THIS SIDE UP"`, `"REFRIGERATE"`.
* The sticker itself doesn't cool or protect the box. But the courier (Spring Framework) reads the sticker and handles the box accordingly.

### How Spring Reads Annotations at Startup:
```
1. You write:           @Service public class PaymentService { ... }
2. Spring Scans:        Spring scans all project files looking for @Service, @Component, etc.
3. Spring Reflection:   Spring uses Reflection to inspect the constructor and fields.
4. Spring IoC:          Spring creates a singleton instance and stores it in the ApplicationContext.
```

---

## 📝 Practice Check
Make sure you can answer these before moving to Module 4:
1. Why is `List<User>` safer than a raw `List`?
2. What problem does `Optional<T>` solve?
3. Why do we prefer `RuntimeException` (Unchecked) in Spring applications?
4. What does an `@Annotation` actually do behind the scenes?
