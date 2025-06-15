# 5. Functions and Modular Code

Functions in Rust are more than just code blocks—they're the building blocks of safe, reusable, and maintainable software.

## 5.1 Function Fundamentals: Beyond the Basics

### Function Declaration and Return Types
```rust
// Explicit return type
fn calculate_tax(amount: f64, rate: f64) -> f64 {
    amount * rate  // No semicolon = implicit return
}

// Unit type (returns nothing)
fn log_message(message: &str) {
    println!("[LOG] {}", message);
}

// Multiple parameters with different types
fn format_user_display(name: &str, age: u32, is_active: bool) -> String {
    format!("User: {} ({}), Status: {}", 
            name, 
            age, 
            if is_active { "Active" } else { "Inactive" })
}
```

### Early Returns and Error Handling
```rust
use std::fs::File;

fn validate_and_process_age(input: &str) -> Result<String, String> {
    // Early return on parse failure
    let age: u32 = input.parse()
        .map_err(|_| "Invalid number format".to_string())?;
    
    // Early return on business logic failure
    if age > 150 {
        return Err("Age seems unrealistic".to_string());
    }
    
    // Success case
    Ok(format!("Age {} is valid", age))
}
```

## 5.2 Advanced Function Patterns

### Function Parameters: Ownership and Borrowing
```rust
// Taking ownership - use when you need to consume the value
fn process_and_consume(data: Vec<i32>) -> i32 {
    data.into_iter().sum()  // data is consumed
}

// Borrowing immutably - use for read-only access
fn calculate_average(numbers: &[i32]) -> f64 {
    if numbers.is_empty() {
        return 0.0;
    }
    numbers.iter().sum::<i32>() as f64 / numbers.len() as f64
}

// Borrowing mutably - use when you need to modify
fn normalize_scores(scores: &mut [f64]) {
    let max_score = scores.iter()
        .max_by(|a, b| a.partial_cmp(b).unwrap())
        .copied()
        .unwrap_or(1.0);
    
    for score in scores.iter_mut() {
        *score /= max_score;
    }
}
```

### Generic Functions: Code Reuse Done Right
```rust
use std::fmt::Display;

// Generic function with trait bounds
fn print_and_return<T: Display + Clone>(value: T) -> T {
    println!("Processing: {}", value);
    value.clone()
}

// Multiple generic parameters
fn find_max<T: PartialOrd + Copy>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

// Generic with where clause for complex bounds
fn process_items<T, F>(items: Vec<T>, processor: F) -> Vec<String>
where
    T: Display,
    F: Fn(&T) -> String,
{
    items.iter()
        .map(|item| processor(item))
        .collect()
}
```

### Higher-Order Functions and Closures
```rust
// Function that takes another function
fn apply_operation<F>(values: &[i32], operation: F) -> Vec<i32>
where
    F: Fn(i32) -> i32,
{
    values.iter().map(|&x| operation(x)).collect()
}

// Using closures with captured variables
fn create_multiplier(factor: i32) -> impl Fn(i32) -> i32 {
    move |x| x * factor  // 'move' transfers ownership to closure
}

// Practical example: Configuration-driven processing
fn process_requests<F>(requests: Vec<Request>, validator: F) -> Vec<Result<Response, String>>
where
    F: Fn(&Request) -> Result<(), String>,
{
    requests.into_iter()
        .map(|req| {
            validator(&req)
                .and_then(|_| process_request(req))
        })
        .collect()
}
```

## 5.3 Module System: Organizing Code for Scale

### Basic Module Structure
```rust
// main.rs or lib.rs
mod network {
    pub mod http {
        pub fn get(url: &str) -> Result<String, String> {
            // HTTP GET implementation
            Ok(format!("Response from {}", url))
        }
        
        pub fn post(url: &str, data: &str) -> Result<String, String> {
            // HTTP POST implementation
            Ok(format!("Posted {} to {}", data, url))
        }
    }
    
    pub mod tcp {
        pub struct Connection {
            address: String,
        }
        
        impl Connection {
            pub fn new(address: String) -> Self {
                Self { address }
            }
            
            pub fn send(&self, data: &str) -> Result<(), String> {
                println!("Sending '{}' to {}", data, self.address);
                Ok(())
            }
        }
    }
}

// Usage
use network::http;
use network::tcp::Connection;

fn main() {
    let response = http::get("https://api.example.com").unwrap();
    
    let conn = Connection::new("127.0.0.1:8080".to_string());
    conn.send("Hello server!").unwrap();
}
```

### File-Based Module Organization
```
src/
├── main.rs
├── lib.rs
├── auth/
│   ├── mod.rs
│   ├── login.rs
│   └── permissions.rs
├── database/
│   ├── mod.rs
│   ├── connection.rs
│   └── migrations.rs
└── api/
    ├── mod.rs
    ├── handlers.rs
    └── middleware.rs
```

```rust
// auth/mod.rs
pub mod login;
pub mod permissions;

pub use login::authenticate_user;
pub use permissions::{Role, check_permission};

// auth/login.rs
use crate::database;

pub fn authenticate_user(username: &str, password: &str) -> Result<User, AuthError> {
    // Authentication logic
    database::find_user_by_credentials(username, password)
}

// main.rs
mod auth;
mod database;
mod api;

use auth::{authenticate_user, Role};

fn main() {
    match authenticate_user("alice", "secret123") {
        Ok(user) => println!("Welcome, {}!", user.name),
        Err(e) => println!("Authentication failed: {:?}", e),
    }
}
```

## 5.4 Error Handling in Functions

### Custom Error Types
```rust
#[derive(Debug)]
enum ProcessingError {
    InvalidInput(String),
    NetworkError(String),
    DatabaseError(String),
}

impl std::fmt::Display for ProcessingError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ProcessingError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
            ProcessingError::NetworkError(msg) => write!(f, "Network error: {}", msg),
            ProcessingError::DatabaseError(msg) => write!(f, "Database error: {}", msg),
        }
    }
}

impl std::error::Error for ProcessingError {}

// Functions that return custom errors
fn validate_email(email: &str) -> Result<(), ProcessingError> {
    if email.contains('@') {
        Ok(())
    } else {
        Err(ProcessingError::InvalidInput(
            format!("'{}' is not a valid email", email)
        ))
    }
}

fn save_user(user: &User) -> Result<u32, ProcessingError> {
    validate_email(&user.email)?;  // Propagate validation error
    
    // Simulated database save
    database::save(user)
        .map_err(|e| ProcessingError::DatabaseError(e.to_string()))
}
```

### Result Combinators and Error Chaining
```rust
fn process_user_registration(data: UserRegistrationData) -> Result<User, ProcessingError> {
    validate_email(&data.email)?;
    
    let user = User::new(data.name, data.email);
    let user_id = save_user(&user)?;
    
    send_welcome_email(&user.email)
        .map_err(|e| ProcessingError::NetworkError(e.to_string()))?;
    
    Ok(User { id: Some(user_id), ..user })
}

// Chaining operations
fn get_user_profile(user_id: u32) -> Result<UserProfile, ProcessingError> {
    database::find_user(user_id)
        .map_err(|e| ProcessingError::DatabaseError(e.to_string()))?
        .and_then(|user| {
            get_user_preferences(user.id)
                .map(|prefs| UserProfile::new(user, prefs))
        })
}
```

## 5.5 Testing Functions and Modules

### Unit Testing
```rust
pub fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_fibonacci_base_cases() {
        assert_eq!(fibonacci(0), 0);
        assert_eq!(fibonacci(1), 1);
    }

    #[test]
    fn test_fibonacci_sequence() {
        assert_eq!(fibonacci(2), 1);
        assert_eq!(fibonacci(3), 2);
        assert_eq!(fibonacci(4), 3);
        assert_eq!(fibonacci(5), 5);
    }

    #[test]
    #[should_panic]
    fn test_invalid_input() {
        validate_email("invalid-email").unwrap();
    }
}
```

### Integration Testing
```rust
// tests/integration_test.rs
use myapp::auth;
use myapp::database;

#[test]
fn test_user_authentication_flow() {
    // Setup test database
    let db = database::setup_test_db();
    
    // Create test user
    let user = database::create_test_user("test@example.com", "password123");
    
    // Test authentication
    let result = auth::authenticate_user("test@example.com", "password123");
    assert!(result.is_ok());
    
    // Cleanup
    database::cleanup_test_db(db);
}
```

## 5.6 Performance and Optimization Patterns

### Zero-Cost Abstractions
```rust
// Generic function that compiles to specific implementations
fn process_data<T, F>(data: &[T], processor: F) -> Vec<T>
where
    T: Clone,
    F: Fn(&T) -> T,
{
    data.iter()
        .map(|item| processor(item))
        .collect()
}

// Usage - the compiler optimizes this to direct operations
let numbers = vec![1, 2, 3, 4, 5];
let doubled = process_data(&numbers, |&x| x * 2);
```

### Avoiding Unnecessary Allocations
```rust
// Instead of returning String, return &str when possible
fn get_error_message(code: u32) -> &'static str {
    match code {
        404 => "Not Found",
        500 => "Internal Server Error",
        _ => "Unknown Error",
    }
}

// Use string formatting only when necessary
fn format_response(status: u32, data: &str) -> String {
    if status == 200 {
        data.to_string()  // Only allocate when needed
    } else {
        format!("Error {}: {}", status, data)
    }
}
```

## 5.7 Real-World Application Architecture

### Layered Architecture Example
```rust
// Domain layer
pub mod domain {
    pub struct User {
        pub id: Option<u32>,
        pub email: String,
        pub name: String,
    }
    
    pub trait UserRepository {
        fn save(&self, user: &User) -> Result<u32, String>;
        fn find_by_email(&self, email: &str) -> Result<Option<User>, String>;
    }
}

// Infrastructure layer
pub mod infrastructure {
    use crate::domain::{User, UserRepository};
    
    pub struct DatabaseUserRepository {
        connection: DatabaseConnection,
    }
    
    impl UserRepository for DatabaseUserRepository {
        fn save(&self, user: &User) -> Result<u32, String> {
            // Database-specific implementation
            Ok(123)
        }
        
        fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
            // Database query implementation
            Ok(None)
        }
    }
}

// Application layer
pub mod application {
    use crate::domain::{User, UserRepository};
    
    pub struct UserService<R: UserRepository> {
        repository: R,
    }
    
    impl<R: UserRepository> UserService<R> {
        pub fn new(repository: R) -> Self {
            Self { repository }
        }
        
        pub fn register_user(&self, email: String, name: String) -> Result<User, String> {
            // Business logic
            let user = User { id: None, email, name };
            let id = self.repository.save(&user)?;
            Ok(User { id: Some(id), ..user })
        }
    }
}
```

## Key Takeaways

1. **Functions are building blocks** - design them for reuse and clarity
2. **Use borrowing wisely** - prefer references over ownership when possible
3. **Generics enable code reuse** - write once, use with many types
4. **Modules provide organization** - separate concerns and control visibility
5. **Error handling is part of design** - use custom error types for clarity
6. **Test your functions** - unit tests catch bugs early
7. **Layer your architecture** - separate business logic from infrastructure

## Practice Projects

1. **Calculator Library**: Build a module with different mathematical operations
2. **Configuration Parser**: Create functions to read and validate config files
3. **HTTP Client**: Design a modular API client with error handling
4. **Task Manager**: Build a todo app with proper separation of concerns

Functions and modules in Rust help you build software that scales—both in terms of performance and maintainability! 