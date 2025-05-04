# Rust Ownership, Borrowing, and Lifetimes: A Practical Primer for Go/Java Devs

## Ownership & Moves

* In Rust, variables own their data by default.
* Passing a value by default **moves** ownership.

```rust
fn consume(s: String) { /* s is dropped here */ }

let a = String::from("hi");
consume(a); // a is now invalid
```

* If you want to retain the original, you must `.clone()` it (which may be expensive).

## Copy vs Clone

* `Copy`: cheap bitwise copy (e.g., primitives)
* `Clone`: explicit heap-aware clone
* Use `#[derive(Copy, Clone)]` for simple structs:

```rust
#[derive(Copy, Clone)]
struct Point(i32, i32);
```

## Borrowing

* Borrow immutable: `&T`
* Borrow mutable: `&mut T`
* Only **one mutable borrow** or **many immutable borrows** allowed at a time

```rust
fn update(x: &mut i32) {
    *x += 1;
}
```

## Lifetimes

* Describes how long a reference is valid
* Functions that return references often need lifetimes:

```rust
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() > b.len() { a } else { b }
}
```

## Lifetime Elision Rules

Rust omits lifetimes in simple cases:

1. Each input gets its own lifetime
2. If one input, the output gets that input's lifetime
3. If `&self`, the return lifetime is that of `self`

When these rules aren’t enough (e.g., multiple refs), annotate explicitly.

## Structs With References

```rust
struct Person<'a> {
    name: &'a str,
}

impl<'a> Person<'a> {
    fn get_name(&self) -> &str {
        self.name
    }
    fn greet<'b>(&'a self, prefix: &'b str) -> &'a str {
        // still returns self.name
        self.name
    }
}
```

## str vs String

| Type     | Owns Data? | Mutable? | Notes                 |
| -------- | ---------- | -------- | --------------------- |
| `str`    | ❌          | ❌        | Unsized UTF-8 view    |
| `&str`   | ❌          | ❌        | Borrowed string slice |
| `String` | ✅          | ✅        | Owned, heap-allocated |

Conversions:

```rust
let s = String::from("hi");
let view: &str = &s;
let clone = s.clone();
```

## Interior Mutability

Used when you want mutation but only have `&self`

### RefCell<T>

* Allows mutation at runtime via borrow checking

```rust
use std::cell::RefCell;

struct Bag {
    count: RefCell<u32>,
}

impl Bag {
    fn increment(&self) {
        *self.count.borrow_mut() += 1;
    }
}
```

### Cell<T>

* Simpler version for `Copy` types

```rust
use std::cell::Cell;
let x = Cell::new(5);
x.set(x.get() + 1);
```

## Borrow Checker Weirdness (Preview)

* Temporaries and scoping rules
* Reborrows (e.g., `let r = &mut *x`)
* Non-lexical lifetimes (NLL) make borrowing more flexible since Rust 2018

---

## Philosophy Recap

* Rust makes you think hard **once**, run safely forever
* Languages like Python/Go optimize for fast writing; Rust optimizes for bug resistance
* Use Rust when correctness, safety, or performance *must* be guaranteed
