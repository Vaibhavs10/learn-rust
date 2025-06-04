# 3. Variables, Data Types, and Mutability

Rust is a statically typed language with powerful type inference and strict rules about mutability.

## 3.1 Variables

Declare a variable with `let`:

```rust
let x = 5;
println!("x = {}", x);
```

Variables are immutable by default. This means you cannot change their value unless you make them mutable.

## 3.2 Mutability

To make a variable mutable, use `mut`:

```rust
let mut y = 10;
println!("y = {}", y);
y = 20;
println!("y = {}", y);
```

## 3.3 Constants

Constants are always immutable and must have a type annotation. Use `const`:

```rust
const MAX_POINTS: u32 = 100_000;
println!("Max points: {}", MAX_POINTS);
```

## 3.4 Shadowing

You can declare a new variable with the same name as a previous one. This is called shadowing:

```rust
let z = 5;
let z = z + 1;
let z = z * 2;
println!("z = {}", z); // z = 12
```

## 3.5 Data Types

Rust has scalar and compound types.

### Scalar Types
- Integers: `i32`, `u32`, `i64`, etc.
- Floating-point: `f32`, `f64`
- Booleans: `bool`
- Characters: `char`

Example:
```rust
let a: i32 = -10;
let b: u32 = 20;
let c: f64 = 3.14;
let d: bool = true;
let e: char = 'R';
```

### Compound Types
- Tuples: group values of different types
- Arrays: fixed-size lists of values

Example:
```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let arr: [i32; 3] = [1, 2, 3];
```

## 3.6 Type Inference

Rust can often infer the type:

```rust
let score = 42; // inferred as i32
let pi = 3.1415; // inferred as f64
```

---

Try changing values, types, and mutability to see how Rust enforces safety! 