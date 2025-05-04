# 🦀 Rust: Stack vs Heap — A Guide for Go/Java Developers

Rust gives you fine-grained control over memory — **unlike Java's heap-only** or **Go's escape analysis**, Rust makes you **explicitly manage** stack vs heap, with **no garbage collector**.

---

## 📚 Stack vs Heap — Basics

| Memory Type | Characteristics                         | Rust Behavior                      |
| ----------- | --------------------------------------- | ---------------------------------- |
| **Stack**   | Fast, limited (\~1MB), LIFO, scoped     | Default for values with known size |
| **Heap**    | Flexible, slower, dynamically allocated | Use `Box`, `Vec`, `String`, etc.   |

---

## ⚠️ Stack Is Limited

```rust
fn main() {
    let big = [0u8; 10_000_000]; // ❌ stack overflow
}
```

Instead, heap-allocate:

```rust
fn main() {
    let big = Box::new([0u8; 10_000_000]); // ✅ heap
}
```

---

## 🧠 Rust Allocation Rules

> **If you write `let x = ...`, it goes on the stack unless you ask otherwise.**

---

## 🔎 Examples

### Scalar

```rust
let a = 42; // i32 — lives entirely on stack
```

---

### Fixed-size Array

```rust
let arr = [1, 2, 3]; // [i32; 3] — stack
```

---

### Dynamic Vector

```rust
let v = vec![1, 2, 3];
// Vec metadata (ptr, len, cap) → stack
// actual elements → heap
```

---

### String

```rust
let s = String::from("hello");
// like Vec<u8>: metadata on stack, data on heap
```

---

### Box

```rust
let b = Box::new(123);
// pointer on stack, 123 on heap
```

---

### Struct Example

```rust
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let p = Person {
        name: String::from("Alice"),
        age: 30,
    };
    // `p` on stack
    // `name`'s string data on heap
}
```

---

## 🧵 Thread Stack Size Control

```rust
use std::thread;

fn main() {
    let handle = thread::Builder::new()
        .stack_size(32 * 1024 * 1024) // 32 MB
        .spawn(|| {
            let _big = [0u8; 16_000_000]; // now safe!
        })
        .unwrap();

    handle.join().unwrap();
}
```

---

## 🧮 Check Size at Compile Time

```rust
use std::mem::size_of;

fn main() {
    println!("Vec<i32>: {}", size_of::<Vec<i32>>()); // ~24 bytes
    println!("Box<i32>: {}", size_of::<Box<i32>>()); // 8 bytes (pointer)
}
```

---

## ✅ Summary: When to Heap

| Situation                    | Use Heap?        |
| ---------------------------- | ---------------- |
| Large array                  | ✅ Box/Vec        |
| Unknown size at compile time | ✅                |
| Recursive structure          | ✅ Box            |
| Want ownership transfer      | Maybe ✅          |
| Small, short-lived value     | ❌ Stick to stack |
