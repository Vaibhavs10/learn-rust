# 2. Hello, Cargo!

Cargo is Rust's build system and package manager. It helps you create, build, and manage Rust projects easily.

## 2.1 Create a New Project

Open your terminal and run:

```sh
cargo new hello_cargo
cd hello_cargo
```

This creates a new directory called `hello_cargo` with the following structure:

```
hello_cargo/
├── Cargo.toml
└── src
    └── main.rs
```

- `Cargo.toml`: Project metadata and dependencies.
- `src/main.rs`: Main source file.

## 2.2 Build and Run Your Project

To build your project:

```sh
cargo build
```

To run it:

```sh
cargo run
```

You should see:
```
   Compiling hello_cargo v0.1.0 (...)
    Finished dev [unoptimized + debuginfo] target(s) in ...
     Running `target/debug/hello_cargo`
Hello, world!
```

## 2.3 Explore the Project Structure

- `src/main.rs` contains your program's entry point:

```rust
fn main() {
    println!("Hello, world!");
}
```

- `Cargo.toml` looks like this:

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
```

You can add dependencies under `[dependencies]`.

## 2.4 Clean and Build Release

To clean build artifacts:

```sh
cargo clean
```

To build an optimized release version:

```sh
cargo build --release
```

This creates a faster executable in `target/release/`.

---

Now you know how to use Cargo to manage your Rust projects! 