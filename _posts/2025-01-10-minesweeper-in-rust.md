---
title: "Minesweeper in Rust"
description: "Implementing a Minesweeper TUI app in Rust, coming from C++"
date: 2025-02-05
published: true
---

Last year, I wrote [nsweeper](github.com/UltimateBoomer/nsweeper), a modern C++ Minesweeper clone.
Since I've been learning Rust recently, and I want to work on a simple project for fun, so I decided to rewrite the game in Rust.

## Design

As for most applications, first I need to decide on the architecture of the project.
I chose to use **model-view-controller** pattern as it is pretty effect for making the project more modular and easier to test.

It makes sense to split each part of the pattern into different modules.
- The model (`src/model/sweeper.rs`) contains structs for stored the Minesweeper game board, stuff used by the timer, and common Minesweeper functions (open tile, flag tile). This is essentially the backend of the program.
- The view (`src/app/sweeper_view.rs`) in this case, contains a function which given a game, returns its text representation which can be displayed on the terminal.
  More on the frontend implementation and library choice later.
- The controller (`src/app/sweeper_controller.rs`), which is also part of the frontend, is a wrapper over the model.
  It allows the game to be started/stopped at any time, and handles cursor manipulation.

The module system in Rust allows each part of the pattern to be separated into different modules.
This also means I can easily replace part of the view or controller in the future if I want to turn this project into a GUI or web app for example.

### Backend

As always my goal is to make the backend simple and generalizable to any frontend.
I decided to use a struct for the game itself, the board, and individual cells with state information.

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

The sweeping logic uses the commonly documented breadth first flood-fill algorithm to reveal all adjacent empty cells when a cell with no adjacent bombs is revealed.

### Frontend

I chose **Ratatui** for the TUI frontend, which is a simple and well designed library that is good enough for displaying the UI elements of the game and handling input.

Similar to my previous project, I decided to use full-width numbers from CJK and emojis to make the numbers take up double character width and close to a square proportion.
This makes the game look close to the real game, as long as the terminal supports unicode well.

The view consists of a single function that takes a game and returns a string representation of the game board.
I thought about encapsulating the view into a struct, but it was not necessary as the view is stateless and doesn't need to store anything.
Instead the controller handles the logistics of the game, including start/stop, difficulty setting and key input.

### Testing

Unit testing is fairly easy in Rust because of its common unit testing tools.
All we need is to define a testing module, which I've put in the same file as the implementations.
However, I'm not sure there is a good way to mock the current time for testing, but that part of the program isn't really too complex or necessary to warrant extensive unit testing.
In my previous experiences with C++ this is usually done by creating my own mocks for time, threads etc. with an abstract interface and concrete/mock implementations, but this requires dynamic dispatch which has a runtime performance penalty.

## What's familiar from C++

After spending a few days on this project, I realized that Rust is not too different from C++ in terms of syntax and features.
For example, `unique_ptr` in C++ is very similar to `Box` in Rust: both are pointers which own the pointed object.
However, there are some differences which make software design a bit different in Rust.

### Move semantics

Modern C++ is known for its useful move semantics.
It allows us to give away and object if it is not useful anymore, which opens the door to optimizations.

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

This sort of mistakes happened many times when I was developing the project, and the solution is to not define references this way.
This does make the code a bit messier coming from C++, but it is a good habit to get into as it reduces the lifetimes requirements of the objects, which can prevent cases of undefined behavior even in C++.

## Next steps

Now that I have a working demo of Minesweeper in Rust, my next steps is to separate the frontend modules and the game model into a different crates, and turn this app into something that can potentially be used in bigger projects.
For example, once I learn more about Rust networking, I can turn the game into a server-client game with a web app.
Another possibility is a multiplayer Minesweeper battle royale like game, which would be a fun project to work on in the future.
