---
title: "Minesweeper in Rust"
date: 2025-01-10
published: false
---

After taking a object-oriented programming course last year, I wrote [nsweeper](github.com/UltimateBoomer/nsweeper), a C++ Minesweeper clone.
Since I've been learning Rust recently, and I want to work on a simple project for fun, I decided to rewrite the game in Rust.

## What's familiar from C++

Reading over the Rust programming language book, what I took away is that Rust has a lot in common with modern C++ in terms of features and the standard library, while the syntax looks closer to something like Kotlin.
For example, `unique_ptr` in C++ is very similar to `Box` in Rust: both are pointers which own the pointed object.
However, there are some differences which make software design a bit different in Rust.

### Move semantics

Modern C++ is known for its useful move semantics.
It allows you to give away and object if it is not useful anymore, which opens the door to optimizations.

```c++
void process_data_copy() {
  vector<int> big_data{}; // size n
  // Do stuff with big_data
  next_step(big_data); // O(n)
  // big_data can be reused
}

void process_data_move() {
  vector<int> big_data{}; // size n
  // Do stuff with big_data
  next_step(std::move(big_data)); // O(1)
  // big_data likely doesn't have data anymore, but is still in a valid state
}
```

In this example, using move means only the data pointer of `big_data` is copied, not the entire data.
However, the object is required to still be in a valid state, so it can still be reused, though I'm not sure how useful that would be.

```rust
fn process_data_move() {
  Vec<int> big_data = Vec::new(); // size n
  // Do stuff with big_data
  next_step(big_data) // O(1)
  // big_data can't be used again, enforced by compiler
}
```

In Rust, if you are passing an object into a function call, the equivalent of move semantics is automatically applied and the object is destroyed.
The object is then actually gone, and can't be used again unless you redefine it.

I think this is a very useful Rust feature to ensure more programs are using move and not randomly performing possibly expensive copies.

### Borrowing

In C++, a habit I tend to do is defining reference variables early in the function, especially if it is going to be used more than once in the function.
This makes sense intuitively as you probably don't want to fetch the same resource or data multiple times in a function.

```c++
void do_stuff() {
  auto resource = get_resource(); // acquire resource
  auto part = resource.get_part(); // get reference to a part of the resource
  process(resource); // use resource again
  return;
  // resource released after scope ends
}
```

While this is fine in C++, in Rust doing this is likely to yield compile errors:

```rust
fn do_stuff() {
  let mut resource = get_resource(); // acquire resource
  let part = &resource.get_part(); // borrow resource as reference
  process(&mut resource); // reference already borrowed as non-mut reference
}
```

Another way this kind of borrowing errors can happen:

```rust
fn do_stuff() {
  let mut resource = get_resource();
  if test(&resource) { // borrow
    process(&mut resource); // double borrowing again
  }
}
```

This happened many times when developing the project, and the solution is to not define reference this way, even if it makes the code look a bit messier.

## Minesweeper design and implementation

I've used the **model-view-controller** pattern again to make the project more modular and make testing easier.

- The model (`src/model/sweeper.rs`) contains structs for stored the Minesweeper game board, stuff used by the timer, and common Minesweeper functions (open tile, flag tile). This is basically the back-end of the program.
- The view (`src/app/sweeper_view.rs`) contains a function which given a game, returns its text representation which can be displayed on the terminal.
  I decided to use full-width numbers from CJK and emojis to make the game look close to the real game, as long as the terminal supports unicode well.
- The controller (`src/app/sweeper_controller.rs`), which is also part of the front-end, is a wrapper over the model.
  It allows the game to be started/stopped at any time, and handles cursor manipulation.

The module system in Rust allows each part of the pattern to be separated into different modules, which keeps coherence high.
This also means I can easily replace part of the view or controller in the future if I want to turn this project into a GUI or web app for example.

Overall the back-end of project boils down to this model:

```rust
struct Cell {
    is_bomb: bool,
    is_flagged: bool,
    is_revealed: bool,
    mine_count: u8,
}

struct Board {
    width: usize,
    height: usize,
    cells: Vec<Cell>,
}

enum GameState {
    NotRunning,
    Running,
    Win,
    Lose,
}

struct SweeperGame {
    board: Board,
    num_bombs: usize,
    num_revealed: usize,
    num_flags: usize,
    state: GameState,
    start_time: Option<Instant>,
    total_time: Duration,
}
```

This allows me to create the necessary functions for minesweeper to work, making the back-end a separate module.

I chose **Ratatui** for the TUI front-end, which is good enough for displaying the UI elements and handling input.

## Testing

Unit testing is fairly easy in Rust because of its unit testing tools.
All we need is to define a testing module, which I've put in the same file as the implementations.
However, I'm not sure there is a good way to mock the current time for testing, but that part of the program isn't really too complex or necessary to warrant extensive unit testing.
In my previous experiences with C++ this is usually done by creating my own mocks for time, threads etc. with an abstract interface and concrete/mock implementations, but this requires dynamic dispatch which has a runtime performance penalty.

## Next steps

Now that I have a working demo of Minesweeper in Rust, I can separate the front-end modules and the game model into a different crates, and turn this app into something that can be used in bigger projects.
