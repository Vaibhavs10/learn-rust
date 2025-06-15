# 4. Control Flow in Action

Control flow in Rust isn't just about branching—it's about expressing logic clearly and handling edge cases safely.

## 4.1 Conditional Logic: Beyond Simple If/Else

### Pattern-Based Conditions
```rust
let user_role = "admin";
let access_level = if user_role == "admin" { 10 } else { 1 };

// Better: Express intent clearly
let access_level = match user_role {
    "admin" => 10,
    "moderator" => 5,
    "user" => 1,
    _ => 0,  // Default for unknown roles
};
```

### Conditional Assignment with Safety
```rust
let temperature = 25;
let status = if temperature > 30 {
    "hot"
} else if temperature < 10 {
    "cold"
} else {
    "comfortable"
};

// Rust ensures all branches return the same type
```

## 4.2 Match: Rust's Superpower

### Exhaustive Pattern Matching
```rust
enum ConnectionState {
    Connected,
    Disconnected,
    Reconnecting,
    Error(String),
}

fn handle_connection(state: ConnectionState) -> String {
    match state {
        ConnectionState::Connected => "All good!".to_string(),
        ConnectionState::Disconnected => "Attempting to reconnect...".to_string(),
        ConnectionState::Reconnecting => "Please wait...".to_string(),
        ConnectionState::Error(msg) => format!("Connection failed: {}", msg),
    }
    // Compiler guarantees we handle every case!
}
```

### Match Guards and Complex Patterns
```rust
fn categorize_score(score: u32) -> &'static str {
    match score {
        90..=100 => "Excellent",
        80..=89 => "Good",
        70..=79 => "Average",
        60..=69 => "Below Average",
        n if n < 60 => "Needs Improvement",
        _ => "Invalid Score",
    }
}

// Destructuring in match
fn process_user_data(data: (String, Option<u32>, bool)) -> String {
    match data {
        (name, Some(age), true) => format!("{} is {} years old and active", name, age),
        (name, Some(age), false) => format!("{} is {} years old but inactive", name, age),
        (name, None, active) => format!("{} age unknown, active: {}", name, active),
    }
}
```

## 4.3 Loops: Iteration Done Right

### For Loops: The Preferred Choice
```rust
// Iterate over collections
let numbers = vec![1, 2, 3, 4, 5];
for num in &numbers {  // Borrow, don't move
    println!("Number: {}", num);
}

// Iterate with indices when needed
for (index, value) in numbers.iter().enumerate() {
    println!("Index {}: {}", index, value);
}

// Range-based iteration
for i in 0..10 {
    println!("Count: {}", i);
}
```

### While Loops: Condition-Based Iteration
```rust
let mut retry_count = 0;
let max_retries = 3;

while retry_count < max_retries {
    match attempt_connection() {
        Ok(_) => break,
        Err(e) => {
            retry_count += 1;
            println!("Attempt {} failed: {}", retry_count, e);
        }
    }
}
```

### Loop with Break Values
```rust
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 5 {
        break counter * 2;  // Return value from loop
    }
};
println!("Result: {}", result);  // Result: 10
```

## 4.4 Advanced Pattern Matching Techniques

### Option and Result Handling
```rust
fn safe_divide(a: f64, b: f64) -> Option<f64> {
    if b != 0.0 { Some(a / b) } else { None }
}

// Pattern matching with Option
match safe_divide(10.0, 2.0) {
    Some(result) => println!("Result: {}", result),
    None => println!("Cannot divide by zero"),
}

// Chaining with if let
if let Some(result) = safe_divide(10.0, 2.0) {
    println!("Division successful: {}", result);
}
```

### Nested Pattern Matching
```rust
struct User {
    name: String,
    email: Option<String>,
    age: Option<u32>,
}

fn describe_user(user: &User) -> String {
    match (&user.email, &user.age) {
        (Some(email), Some(age)) => {
            format!("{} ({}) - Contact: {}", user.name, age, email)
        }
        (Some(email), None) => {
            format!("{} - Contact: {}", user.name, email)
        }
        (None, Some(age)) => {
            format!("{} ({})", user.name, age)
        }
        (None, None) => user.name.clone(),
    }
}
```

## 4.5 Error Handling Patterns

### Early Returns with Question Mark
```rust
use std::fs::File;
use std::io::Read;

fn read_config_file() -> Result<String, std::io::Error> {
    let mut file = File::open("config.txt")?;  // Early return on error
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;       // Another early return
    Ok(contents)
}

// Handling the result
match read_config_file() {
    Ok(config) => println!("Config loaded: {}", config),
    Err(e) => eprintln!("Failed to load config: {}", e),
}
```

### Match vs If Let: Choosing Your Tool
```rust
// Use match for exhaustive handling
match user_input {
    "start" => start_process(),
    "stop" => stop_process(),
    "pause" => pause_process(),
    _ => show_help(),
}

// Use if let for single-case handling
if let Some(first_item) = items.first() {
    process_item(first_item);
}
```

## 4.6 Practical Patterns: State Machines

```rust
#[derive(Debug)]
enum TaskState {
    Pending,
    InProgress { started_at: u64 },
    Completed { duration: u64 },
    Failed { error: String },
}

impl TaskState {
    fn transition(&mut self, event: TaskEvent) {
        *self = match (self, event) {
            (TaskState::Pending, TaskEvent::Start(time)) => {
                TaskState::InProgress { started_at: time }
            }
            (TaskState::InProgress { started_at }, TaskEvent::Complete(end_time)) => {
                TaskState::Completed { duration: end_time - started_at }
            }
            (TaskState::InProgress { .. }, TaskEvent::Fail(error)) => {
                TaskState::Failed { error }
            }
            _ => return,  // Invalid transition, ignore
        };
    }
}

enum TaskEvent {
    Start(u64),
    Complete(u64),
    Fail(String),
}
```

## 4.7 Performance Considerations

### Iterator Chains vs Loops
```rust
let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Functional approach - often faster due to optimizations
let sum: i32 = numbers
    .iter()
    .filter(|&&x| x % 2 == 0)
    .map(|&x| x * x)
    .sum();

// Traditional loop approach
let mut sum = 0;
for &num in &numbers {
    if num % 2 == 0 {
        sum += num * num;
    }
}
```

## Key Takeaways

1. **Match is exhaustive** - the compiler ensures you handle all cases
2. **Pattern matching prevents bugs** - no forgotten edge cases
3. **Early returns** with `?` make error handling clean
4. **For loops are preferred** over manual indexing
5. **State machines** with enums create robust, maintainable code
6. **Iterator chains** are both expressive and performant

Control flow in Rust guides you toward writing code that's both safe and expressive—the compiler won't let you forget edge cases!

## Practice Exercises

1. **Traffic Light State Machine**: Create an enum for traffic light states and implement transitions
2. **User Authentication Flow**: Use match to handle different authentication states
3. **File Processing Pipeline**: Chain iterators to process and filter file data
4. **Retry Logic**: Implement exponential backoff using loops and error handling

Try building these patterns—they're the foundation of robust Rust applications! 