# 3. Variables, Data Types, and Mutability

Rust's approach to variables isn't just different—it prevents entire categories of bugs at compile time.

## 3.1 Variable Declaration and Core Types

Understanding Rust's type system is crucial for writing safe, efficient code.

### Variable Declaration Patterns
```rust
let name = "Alice";              // Type inferred as &str
let age: u32 = 25;              // Explicit type annotation
let mut score = 0;              // Mutable variable
let (x, y) = (10, 20);          // Destructuring assignment
```

### Scalar Types: The Building Blocks

**Integers** - Choose based on your domain:
```rust
let user_id: u32 = 12345;       // Always positive, up to 4.2 billion
let temperature: i16 = -40;     // Can be negative, -32k to +32k
let file_size: u64 = 1_048_576; // Large positive numbers
let offset: isize = -10;        // Pointer-sized, platform dependent
```

**Floating Point** - When precision matters:
```rust
let price: f32 = 19.99;         // 32-bit, sufficient for most decimals
let scientific: f64 = 6.022e23; // 64-bit, high precision calculations
```

**Booleans and Characters**:
```rust
let is_authenticated: bool = true;
let grade: char = 'A';          // Unicode scalar, 4 bytes
let emoji: char = '🦀';         // Rust crab!
```

### Compound Types: Grouping Data

**Tuples** - Fixed-size, mixed types:
```rust
let person: (String, u32, bool) = ("Bob".to_string(), 30, true);
let rgb: (u8, u8, u8) = (255, 0, 128);  // Color values 0-255
let coordinates = (12.5, -3.8);         // Inferred as (f64, f64)
```

**Arrays** - Fixed-size, same type:
```rust
let weekdays: [&str; 5] = ["Mon", "Tue", "Wed", "Thu", "Fri"];
let buffer: [u8; 1024] = [0; 1024];     // Initialize 1024 zeros
let primes = [2, 3, 5, 7, 11];          // Inferred as [i32; 5]
```

### String Types: Text Handling Done Right

```rust
let literal: &str = "Hello";            // String slice, immutable
let owned: String = "World".to_string(); // Owned string, growable
let formatted = format!("{}!", owned);   // Create new String
```

### Collection Types: Dynamic Data

```rust
let mut numbers: Vec<i32> = vec![1, 2, 3];  // Growable array
let mut scores = Vec::new();                // Empty vector, type inferred later
scores.push(95);                           // Now Vec<i32>

use std::collections::HashMap;
let mut users: HashMap<u32, String> = HashMap::new();
users.insert(1, "Alice".to_string());
```

### Reference Types: Borrowing Without Owning

```rust
let data = vec![1, 2, 3, 4, 5];
let slice: &[i32] = &data[1..4];        // Borrow part of the vector
let first: &i32 = &data[0];             // Borrow single element
```

## 3.2 Immutability by Default: Why It Matters

```rust
let connection_count = 0;
// connection_count = 1; // ❌ Compile error!
```

This isn't restrictive—it's **protective**. Immutable variables prevent accidental modifications that cause bugs in concurrent code, make reasoning about program state easier, and enable compiler optimizations.

**When you need change, be explicit:**
```rust
let mut active_users = Vec::new();
active_users.push("alice");  // ✅ Clear intent
```

## 3.3 Shadowing vs Mutation: Choose Your Tool

Shadowing creates a **new variable** with the same name:
```rust
let data = "42";           // String
let data = data.parse::<i32>().unwrap();  // i32
// Type transformation without mutation
```

Mutation changes an **existing variable**:
```rust
let mut counter = 0;
counter += 1;  // Same variable, new value
```

**Rule of thumb:** Use shadowing for type/value transformations, mutation for accumulating changes.

## 3.4 Smart Type System in Action

Rust infers types but catches mismatches:
```rust
let user_id = 42;           // i32 by default
let scores = vec![85, 92, 78];  // Vec<i32>

// This won't compile - prevents subtle bugs
// scores.push("invalid");  // ❌ Type mismatch
```

**Be explicit when precision matters:**
```rust
let temperature: f64 = 98.6;  // Medical precision
let timeout_ms: u64 = 5000;   // No negative timeouts
```

## 3.5 Zero-Cost Abstractions with Compound Types

**Tuples** for related data without overhead:
```rust
fn get_user_info() -> (String, u32, bool) {
    ("Alice".to_string(), 25, true)  // name, age, is_active
}

let (name, age, _) = get_user_info();  // Destructure what you need
```

**Arrays** for known-size collections:
```rust
let daily_temps: [f64; 7] = [20.1, 22.5, 19.8, 21.2, 23.0, 24.1, 20.9];
// Stack-allocated, no heap allocation overhead
```

## 3.6 Constants: More Than Just Immutable

Constants are computed at **compile time** and have global scope:
```rust
const MAX_CONNECTIONS: usize = 1000;
const ERROR_MESSAGE: &str = "Connection failed";

// Use for configuration, lookup tables, magic numbers
const HTTP_OK: u16 = 200;
```

## 3.7 Practical Pattern: Safe State Management

```rust
struct Config {
    max_retries: u32,
    timeout_ms: u64,
}

impl Config {
    fn new() -> Self {
        Self {
            max_retries: 3,      // Immutable after creation
            timeout_ms: 5000,
        }
    }
    
    // Controlled mutation through methods
    fn with_timeout(mut self, ms: u64) -> Self {
        self.timeout_ms = ms;
        self
    }
}
```

## 3.8 Putting It All Together: A Shopping Cart Example

Let's see how data types and mutability work in a practical scenario:

```rust
fn main() {
    // Immutable user data - won't change during session
    let user_name = "Alice";
    let user_id: u32 = 12345;
    let is_premium: bool = true;
    
    // Mutable shopping cart - will change as user shops
    let mut cart_items: Vec<String> = Vec::new();
    let mut total_price: f64 = 0.0;
    let mut item_count: u32 = 0;
    
    // Adding items to cart (mutation)
    cart_items.push("Laptop".to_string());
    cart_items.push("Mouse".to_string());
    cart_items.push("Keyboard".to_string());
    
    // Updating totals (mutation)
    total_price += 999.99;  // Laptop
    total_price += 25.50;   // Mouse  
    total_price += 79.99;   // Keyboard
    item_count = cart_items.len() as u32;
    
    // Apply discount using immutable data
    let discount_rate: f64 = if is_premium { 0.10 } else { 0.05 };
    let discount_amount = total_price * discount_rate;
    let final_price = total_price - discount_amount;
    
    // Display order summary (using compound types)
    let order_summary = (user_id, item_count, final_price);
    println!("User: {} (ID: {})", user_name, order_summary.0);
    println!("Items: {}", order_summary.1);
    println!("Total: ${:.2}", order_summary.2);
    
    // Shadowing for data transformation
    let final_price = format!("${:.2}", final_price);  // f64 → String
    println!("Formatted price: {}", final_price);
}
```

**What this example shows:**
- **Immutable variables** (`user_name`, `user_id`) for data that shouldn't change
- **Mutable variables** (`cart_items`, `total_price`) for state that evolves
- **Different data types** working together (strings, numbers, booleans, vectors)
- **Type annotations** where clarity matters (`u32`, `f64`)
- **Compound types** (tuple) for grouping related data
- **Shadowing** for safe type transformations
- **Safe arithmetic** with explicit type handling

The compiler ensures you can't accidentally modify `user_name` or forget to mark `cart_items` as mutable—preventing bugs before they happen!

## Key Takeaways

1. **Immutability prevents bugs** - especially in concurrent code
2. **Explicit mutability** makes change obvious in code reviews
3. **Type inference** reduces boilerplate while maintaining safety
4. **Shadowing** enables clean data transformations
5. **Choose the right type** for your domain (u32 for counts, f64 for precision)

The compile-time guarantees aren't restrictions—they're your safety net for building reliable software. 