# 4. Control Flow in Action

Control flow determines how your program makes decisions and repeats actions. In Rust, control flow isn't just about branching and looping—it's about expressing logic safely, handling all possible cases, and letting the compiler catch your mistakes before they become bugs.

## What Makes Rust's Control Flow Special?

Unlike many languages, Rust's control flow constructs are:
- **Exhaustive**: The compiler ensures you handle every possible case
- **Expression-based**: Most control flow returns values, not just executes statements  
- **Memory-safe**: No null pointer dereferences or buffer overflows
- **Zero-cost**: The abstractions compile down to efficient machine code

Think of Rust's control flow as having a very strict, helpful teacher who won't let you turn in incomplete work—and that's exactly what makes your programs robust.

## The Control Flow Mindset

Before diving into syntax, understand the philosophy:

1. **Make invalid states unrepresentable** - Use enums and match to model your problem domain
2. **Let the compiler catch your mistakes** - Embrace exhaustive matching instead of fighting it
3. **Express intent, not implementation** - Use iterators to say "what" not "how"
4. **Handle errors explicitly** - No hidden nulls or exceptions to surprise you later

These principles will guide you to writing Rust code that's not just correct, but maintainable and bug-free.

## 4.1 Conditional Logic: Beyond Simple If/Else

### The Foundation: If Expressions
```rust
// In Rust, if is an expression that returns a value
let temperature = 25;
let clothing = if temperature > 20 { "t-shirt" } else { "jacket" };
```

The key insight: Rust's `if` always returns a value, and both branches must return the same type. This prevents entire classes of bugs.

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

The `match` expression is where Rust truly shines. It's not just a switch statement—it's a powerful pattern matching system that:
- Forces you to handle every possible case (exhaustive matching)
- Extracts data from complex types safely
- Compiles to highly efficient code
- Makes impossible states unrepresentable

### Why Match Instead of If/Else Chains?

```rust
// Fragile: Easy to forget cases or make logic errors
let status_message = if user_role == "admin" {
    "Full access"
} else if user_role == "moderator" {
    "Limited access"
} else if user_role == "user" {
    "Basic access"
} else {
    "No access"  // What if we add a new role and forget to update this?
};

// Robust: Compiler ensures we handle every case
enum UserRole {
    Admin,
    Moderator,
    User,
    Guest,
}

let status_message = match user_role {
    UserRole::Admin => "Full access",
    UserRole::Moderator => "Limited access", 
    UserRole::User => "Basic access",
    UserRole::Guest => "Read-only access",
    // If we add a new role, this won't compile until we handle it!
};
```

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

Rust's approach to loops prioritizes safety and expressiveness. Instead of manual index manipulation (a common source of bugs), Rust encourages iterator-based loops that are both safer and often faster.

### The Rust Way: Iterators Over Indices

```rust
let numbers = vec![1, 2, 3, 4, 5];

// ❌ C-style: Error-prone, verbose
for i in 0..numbers.len() {
    println!("Number: {}", numbers[i]);  // Could panic if index is wrong!
}

// ✅ Rust-style: Safe, clear intent
for num in &numbers {
    println!("Number: {}", num);  // Cannot go out of bounds
}
```

The iterator approach eliminates:
- Index out of bounds errors
- Off-by-one mistakes  
- Complexity of manual index management

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

## 4.8 Real-World Example: Simple Todo App

Let's build a basic todo application that demonstrates the key control flow concepts in a practical way.

```rust
// 1. Model the problem with enums - making invalid states impossible
#[derive(Debug, Clone, PartialEq)]
enum TaskStatus {
    Todo,
    Done,
}

#[derive(Debug)]
struct Task {
    id: u32,
    title: String,
    status: TaskStatus,
}

// 2. Simple error handling
#[derive(Debug)]
enum TodoError {
    TaskNotFound(u32),
    EmptyTitle,
}

// 3. Main todo manager
struct TodoApp {
    tasks: Vec<Task>,
    next_id: u32,
}

impl TodoApp {
    fn new() -> Self {
        Self {
            tasks: Vec::new(),
            next_id: 1,
        }
    }

    // Creating tasks with validation
    fn add_task(&mut self, title: String) -> Result<u32, TodoError> {
        // Early return pattern - check for empty title
        if title.trim().is_empty() {
            return Err(TodoError::EmptyTitle);
        }

        let task = Task {
            id: self.next_id,
            title,
            status: TaskStatus::Todo,
        };

        self.tasks.push(task);
        let id = self.next_id;
        self.next_id += 1;
        Ok(id)
    }

    // Pattern matching for state changes
    fn toggle_task(&mut self, task_id: u32) -> Result<(), TodoError> {
        // Find the task using iterator patterns
        let task = self.tasks
            .iter_mut()
            .find(|task| task.id == task_id)
            .ok_or(TodoError::TaskNotFound(task_id))?;

        // Match on current status to toggle
        task.status = match task.status {
            TaskStatus::Todo => TaskStatus::Done,
            TaskStatus::Done => TaskStatus::Todo,
        };

        Ok(())
    }

    // Using iterators to filter and count
    fn get_summary(&self) -> String {
        let total = self.tasks.len();
        let completed = self.tasks
            .iter()
            .filter(|task| task.status == TaskStatus::Done)
            .count();
        let remaining = total - completed;

        // Conditional logic for user-friendly messages
        match (total, remaining) {
            (0, _) => "No tasks yet! Add some to get started.".to_string(),
            (_, 0) => "🎉 All tasks completed! Great job!".to_string(),
            (_, 1) => "1 task remaining".to_string(),
            (_, n) => format!("{} tasks remaining", n),
        }
    }

    // Display all tasks using iteration and pattern matching
    fn display_tasks(&self) {
        if self.tasks.is_empty() {
            println!("No tasks to display.");
            return;
        }

        println!("📋 Your Tasks:");
        for task in &self.tasks {
            let status_icon = match task.status {
                TaskStatus::Todo => "⏳",
                TaskStatus::Done => "✅",
            };
            println!("  {} {} (ID: {})", status_icon, task.title, task.id);
        }
        println!("\n{}", self.get_summary());
    }
}

// 4. Demo function showing everything working together
fn demo_todo_app() {
    let mut app = TodoApp::new();

    // Add some tasks
    match app.add_task("Learn Rust".to_string()) {
        Ok(id) => println!("✅ Added task with ID: {}", id),
        Err(e) => println!("❌ Failed to add task: {:?}", e),
    }

    app.add_task("Write code".to_string()).unwrap();
    app.add_task("Deploy app".to_string()).unwrap();

    // Display initial state
    app.display_tasks();

    // Complete some tasks
    println!("\n📝 Completing first task...");
    if let Err(e) = app.toggle_task(1) {
        println!("Error: {:?}", e);
    }

    app.display_tasks();

    // Try to complete non-existent task (error handling)
    println!("\n🚫 Trying to toggle non-existent task...");
    match app.toggle_task(999) {
        Ok(_) => println!("Task toggled successfully"),
        Err(TodoError::TaskNotFound(id)) => println!("Task {} not found!", id),
        Err(e) => println!("Other error: {:?}", e),
    }
}

// Usage: demo_todo_app();
```

### What This Simplified Example Shows:

1. **Enum for State**: `TaskStatus` prevents invalid states (no "maybe done" tasks!)
2. **Pattern Matching**: 
   - `match` for toggling task status
   - `match` for generating user-friendly messages
3. **Error Handling**: Simple `Result` types with meaningful errors
4. **Iterator Patterns**: Using `.filter()` and `.find()` instead of manual loops
5. **Conditional Logic**: `if/else` and `match` for different scenarios

### Key Control Flow Patterns:

- **Early Returns**: Check for errors first, continue with valid data
- **Exhaustive Matching**: Handle every possible task status
- **Safe Iteration**: No index errors, just expressive operations
- **Error Propagation**: Use `?` and `match` to handle failures gracefully

This example is much simpler but still demonstrates how Rust's control flow patterns work together to create reliable, maintainable code. The compiler ensures we handle all cases and catch errors early!

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