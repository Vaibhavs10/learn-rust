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

## 4.8 Real-World Example: Task Management System

Let's build a simple task management system that demonstrates all the control flow concepts we've learned. This example shows how these patterns work together in a practical application.

```rust
use std::collections::HashMap;
use std::fmt;

// 1. Model the domain with enums (making invalid states impossible)
#[derive(Debug, Clone, PartialEq)]
enum TaskStatus {
    Todo,
    InProgress { assigned_to: String },
    Completed { completed_by: String },
    Cancelled { reason: String },
}

#[derive(Debug, Clone)]
struct Task {
    id: u32,
    title: String,
    description: String,
    status: TaskStatus,
    priority: Priority,
}

#[derive(Debug, Clone, PartialEq)]
enum Priority {
    Low,
    Medium,
    High,
    Critical,
}

// 2. Custom error type for our domain
#[derive(Debug)]
enum TaskError {
    TaskNotFound(u32),
    InvalidTransition { from: TaskStatus, to: TaskStatus },
    EmptyTitle,
    UserNotFound(String),
}

impl fmt::Display for TaskError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            TaskError::TaskNotFound(id) => write!(f, "Task {} not found", id),
            TaskError::InvalidTransition { from, to } => {
                write!(f, "Cannot transition from {:?} to {:?}", from, to)
            }
            TaskError::EmptyTitle => write!(f, "Task title cannot be empty"),
            TaskError::UserNotFound(user) => write!(f, "User '{}' not found", user),
        }
    }
}

// 3. Main task manager using all our control flow patterns
struct TaskManager {
    tasks: HashMap<u32, Task>,
    next_id: u32,
    users: Vec<String>,
}

impl TaskManager {
    fn new() -> Self {
        Self {
            tasks: HashMap::new(),
            next_id: 1,
            users: vec!["Alice".to_string(), "Bob".to_string(), "Charlie".to_string()],
        }
    }

    // Pattern matching with Result for error handling
    fn create_task(&mut self, title: String, description: String, priority: Priority) -> Result<u32, TaskError> {
        // Early return pattern with ?
        if title.trim().is_empty() {
            return Err(TaskError::EmptyTitle);
        }

        let task = Task {
            id: self.next_id,
            title,
            description,
            status: TaskStatus::Todo,
            priority,
        };

        self.tasks.insert(self.next_id, task);
        let id = self.next_id;
        self.next_id += 1;
        Ok(id)
    }

    // Exhaustive pattern matching for state transitions
    fn update_task_status(&mut self, task_id: u32, new_status: TaskStatus) -> Result<(), TaskError> {
        let task = self.tasks.get_mut(&task_id).ok_or(TaskError::TaskNotFound(task_id))?;
        
        // Validate transition using match
        match (&task.status, &new_status) {
            // Valid transitions
            (TaskStatus::Todo, TaskStatus::InProgress { assigned_to }) => {
                if !self.users.contains(assigned_to) {
                    return Err(TaskError::UserNotFound(assigned_to.clone()));
                }
            }
            (TaskStatus::InProgress { .. }, TaskStatus::Completed { .. }) => {}
            (TaskStatus::Todo, TaskStatus::Cancelled { .. }) => {}
            (TaskStatus::InProgress { .. }, TaskStatus::Cancelled { .. }) => {}
            
            // Invalid transitions
            (from, to) => {
                return Err(TaskError::InvalidTransition {
                    from: from.clone(),
                    to: to.clone(),
                });
            }
        }

        task.status = new_status;
        Ok(())
    }

    // Iterator patterns for filtering and processing
    fn get_tasks_by_status(&self, status: TaskStatus) -> Vec<&Task> {
        self.tasks
            .values()
            .filter(|task| std::mem::discriminant(&task.status) == std::mem::discriminant(&status))
            .collect()
    }

    fn get_high_priority_tasks(&self) -> Vec<&Task> {
        self.tasks
            .values()
            .filter(|task| matches!(task.priority, Priority::High | Priority::Critical))
            .collect()
    }

    // Complex business logic using multiple control flow patterns
    fn process_daily_tasks(&mut self) -> Result<String, TaskError> {
        let mut report = String::new();
        let mut processed_count = 0;

        // Iterate over all tasks
        for task in self.tasks.values() {
            // Pattern match on status and priority
            match (&task.status, &task.priority) {
                (TaskStatus::Todo, Priority::Critical) => {
                    report.push_str(&format!("⚠️  URGENT: {} (ID: {})\n", task.title, task.id));
                    processed_count += 1;
                }
                (TaskStatus::InProgress { assigned_to }, Priority::High) => {
                    report.push_str(&format!("🔥 High priority task '{}' assigned to {}\n", 
                                           task.title, assigned_to));
                    processed_count += 1;
                }
                (TaskStatus::Completed { completed_by }, _) => {
                    report.push_str(&format!("✅ Completed: {} (by {})\n", 
                                           task.title, completed_by));
                    processed_count += 1;
                }
                _ => {} // Handle other cases silently
            }
        }

        // Conditional logic for report summary
        let summary = if processed_count == 0 {
            "No tasks requiring attention today.".to_string()
        } else if processed_count == 1 {
            "1 task processed.".to_string()
        } else {
            format!("{} tasks processed.", processed_count)
        };

        Ok(format!("{}\n📊 Summary: {}", report, summary))
    }
}

// 4. Demonstration function showing the system in action
fn demonstrate_task_manager() -> Result<(), TaskError> {
    let mut manager = TaskManager::new();

    // Create some tasks
    let task1 = manager.create_task(
        "Fix critical bug".to_string(),
        "System crashes on startup".to_string(),
        Priority::Critical,
    )?;

    let task2 = manager.create_task(
        "Write documentation".to_string(),
        "Update API docs".to_string(),
        Priority::Medium,
    )?;

    let task3 = manager.create_task(
        "Code review".to_string(),
        "Review PR #123".to_string(),
        Priority::High,
    )?;

    // Update task statuses using pattern matching
    manager.update_task_status(task1, TaskStatus::InProgress { 
        assigned_to: "Alice".to_string() 
    })?;

    manager.update_task_status(task2, TaskStatus::Completed { 
        completed_by: "Bob".to_string() 
    })?;

    // Generate daily report
    let report = manager.process_daily_tasks()?;
    println!("{}", report);

    // Demonstrate error handling
    match manager.update_task_status(999, TaskStatus::Todo) {
        Ok(_) => println!("Should not reach here"),
        Err(e) => println!("Expected error: {}", e),
    }

    Ok(())
}

// Usage: demonstrate_task_manager().unwrap();
```

### What This Example Demonstrates:

1. **Enum Modeling**: `TaskStatus` and `Priority` make invalid states impossible
2. **Pattern Matching**: Exhaustive `match` statements for state transitions
3. **Error Handling**: Custom error types with `Result` and `?` operator
4. **Iterator Patterns**: Filtering and processing collections safely
5. **Conditional Logic**: `if/else` for business logic decisions
6. **Complex Control Flow**: Multiple patterns working together in `process_daily_tasks()`

### Key Patterns in Action:

- **State Validation**: The `update_task_status` method uses `match` to ensure only valid transitions happen
- **Error Propagation**: The `?` operator handles errors cleanly throughout
- **Safe Iteration**: No manual indexing, just expressive iterator chains
- **Exhaustive Handling**: Every possible state is explicitly handled

This is how control flow patterns scale from simple examples to real applications—by composing these safe, expressive constructs into robust business logic.

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