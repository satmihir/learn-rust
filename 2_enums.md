# Rust Enums — A Practical Primer for Go/Java Developers

Rust enums are far more powerful than their Java or Go counterparts. They are **algebraic data types (ADTs)** that let you model a value that can be *one of several different forms*, each potentially carrying different data.

---

## 1. Enums Are Sum Types

A sum type means the value can be *one of many variants*, each with its own data.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}
```

Each variant can carry different kinds of data — unlike Java/Go enums which are just named constants.

---

## 2. Pattern Matching with `match`

```rust
fn process(msg: Message) {
    match msg {
        Message::Quit => println!("Quit"),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Text: {}", text),
        Message::ChangeColor(r, g, b) => println!("Color: {}, {}, {}", r, g, b),
    }
}
```

Rust enforces *exhaustive matching*, so you don’t forget to handle a case.

---

## 3. Option and Result

Rust replaces `null` and exceptions with enums:

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Option Example

```rust
let maybe_name: Option<&str> = Some("Mihir");

match maybe_name {
    Some(name) => println!("Name is {}", name),
    None => println!("No name"),
}
```

### Result Example

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("division by zero".to_string())
    } else {
        Ok(a / b)
    }
}
```

---

## 4. Enum with Impl

```rust
enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(r) => std::f64::consts::PI * r * r,
            Shape::Rectangle(w, h) => w * h,
        }
    }

    fn describe(&self) {
        match self {
            Shape::Circle(r) => println!("Circle of radius {}", r),
            Shape::Rectangle(w, h) => println!("Rectangle of size {} x {}", w, h),
        }
    }

    fn new_rectangle(w: f64, h: f64) -> Self {
        Shape::Rectangle(w, h)
    }
}
```

---

## 5. Struct-like Variants

```rust
struct Point { x: f64, y: f64 }

enum Shape {
    Circle { center: Point, radius: f64 },
    Rectangle { top_left: Point, width: f64, height: f64 },
}
```

Usage:

```rust
let s = Shape::Circle { center: Point { x: 0.0, y: 0.0 }, radius: 3.0 };
```

Pattern Matching:

```rust
match s {
    Shape::Circle { center, radius } => println!("r = {}, center = ({}, {})", radius, center.x, center.y),
    Shape::Rectangle { width, height, .. } => println!("{}x{}", width, height),
}
```

---

## 6. Shorthand: `if let` and `while let`

```rust
if let Some(name) = maybe_name {
    println!("Hello, {}", name);
}

while let Some(x) = stack.pop() {
    println!("{}", x);
}
```

---

## 7. Nested Enums and Match Guards

```rust
enum Status { Active, Inactive }

enum User {
    LoggedIn { name: String, status: Status },
    Guest,
}

match user {
    User::LoggedIn { name, status: Status::Active } => println!("{} is active", name),
    User::LoggedIn { name, .. } => println!("{} is not active", name),
    User::Guest => println!("Guest user"),
}

match num {
    Some(n) if n > 10 => println!("Big number"),
    Some(n) => println!("Small: {}", n),
    None => println!("No value"),
}
```

---

## 8. Recursive Enum: AST Evaluator

```rust
enum Expr {
    Num(i32),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
}

impl Expr {
    fn eval(&self) -> i32 {
        match self {
            Expr::Num(n) => *n,
            Expr::Add(l, r) => l.eval() + r.eval(),
            Expr::Mul(l, r) => l.eval() * r.eval(),
        }
    }
}

let expr = Expr::Add(
    Box::new(Expr::Num(2)),
    Box::new(Expr::Mul(Box::new(Expr::Num(3)), Box::new(Expr::Num(4)))),
);

println!("Result: {}", expr.eval()); // 14
```

---

## 9. Tips and Extras

* Derive traits like `Debug`, `Clone`, `PartialEq`:

  ```rust
  #[derive(Debug, Clone, PartialEq)]
  enum Direction { North, South, East, West }
  ```

* `Option<Box<T>>` is optimized to be just a pointer in size — thanks to *null-pointer optimization*.

* Enums can be generic:

  ```rust
  enum Wrapper<T> {
      One(T),
      Two(T, T),
  }
  ```

---

Rust enums = tagged unions + methods + pattern matching + compiler safety. Way more than `enum { RED, GREEN }`!

---

**Next Steps:** Look into `enum repr`, memory layout, and how enums integrate with async/await (`Poll`, `Future`, etc.)
