# 1. Setting Up Your Rust Development Environment

Welcome to Rust! Let's get your environment ready so you can start coding right away.

## 1.1 Install Rust

Rust provides an installer called `rustup` that sets up everything you need.

### On macOS/Linux:
```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### On Windows:
Download and run [rustup-init.exe](https://rustup.rs/).

Follow the prompts to complete the installation. After installation, restart your terminal and check your Rust version:

```sh
rustc --version
```

You should see something like:
```
rustc 1.XX.X (xxxxxxx YYYY-MM-DD)
```

## 1.2 Set Up Your Editor

Rust works well with many editors. Popular choices:
- **VS Code**: Install the [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) extension.
- **IntelliJ IDEA**: Use the [Rust plugin](https://plugins.jetbrains.com/plugin/8182-rust).
- **Vim/Neovim**: Use plugins like [rust.vim](https://github.com/rust-lang/rust.vim) and [coc-rust-analyzer](https://github.com/fannheyward/coc-rust-analyzer).

## 1.3 Run Your First Rust Program

Let's make sure everything works by running a simple program.

1. Open a terminal.
2. Run:

```sh
rustc --version
```

3. Create a file called `main.rs` with this content:

```rust
fn main() {
    println!("Hello, Rust!");
}
```

4. Compile and run it:

```sh
rustc main.rs
./main
```

You should see:
```
Hello, Rust!
```

Congratulations! Your Rust environment is ready. 