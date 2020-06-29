+++
title = "First Macro: timeit!"
date = 2020-07-03
author = "Mat"
in_search_index = true

[taxonomies]
tags = ["rust"]
+++

This series serves as a practical (*but not-exhaustive*) introduction to declarative `macro_rules!`. I've put together some [Rust macro examples](https://github.com/thepacketgeek/rust-macros-demo) to show how macros can be helpful for improving ergonomics around repetitive or error-prone tasks. The examples cover some great scenarios for macros like:

- Print out the time a block of code takes to execute
- Adding retries around intermittently fallible code
- Making data structure initialization easier for common options
<!-- more -->

## What are macros?
To introduce macros, I'll take a direct quote from the [Macro chapter in the Rust Book](https://doc.rust-lang.org/1.7.0/book/macros.html):

> Macros allow us to abstract at a syntactic level. A macro invocation is shorthand for an "expanded" syntactic form. This expansion happens early in compilation, before any static checking. As a result, macros can capture many patterns of code reuse that Rust’s core abstractions cannot.

The example given shows how `vec![]`, which you're likely used, is a shorthand alternative to a tedious `Vec` instantiation:

```rust
let my_values: Vec<u3> = vec![10, 20, 30, 40];
```

becomes:
```rust
let my_values = {
    let mut temp_vec: Vec<u32> = Vec::new();
    temp_vec.push(10);
    temp_vec.push(20);
    temp_vec.push(30);
    temp_vec.push(40);
    temp_vec
}
```

What we'll cover in this post is how we can use `macro_rules!` to create our own shorthand abstractions.

### Viewing macro output
As we start building some macros I'll be showing output of the generated Rust code using [cargo-expand](https://crates.io/crates/cargo-expand). It's an invaluable tool to see what the `macro_rules!` code is generating. I recommend installing `cargo-expand` to be able to follow along and test out your own macros.

### Defining Macros with macro_rules!
Rust has a special syntax for declaring macros via `macro_rules!` blocks. Macros will have one or more match rules, to allow the macro to match & parse the tokens & expressions withing the invoking call. In that way, you'll see some similarities to Rust's `match` blocks and value destructuring. Let's look at how the syntax is laid out:

```rust
/* |--- Tell Rust compiler: this is a macro definition
   v           v-- macro name (how it will be called)  */
macro_rules! macro_name {
/*   |--- Match rule
     v          v -- macro name (how it will be called)  */
    (...) => { ... }
    // more match rules
}
```

To see the full details on macro syntax, check out the [Rust Book's Syntax section](https://doc.rust-lang.org/1.7.0/book/macros.html#syntactic-requirements). The rest of this post will cover just a subset of topics.

## Building timeit!()

The rest of this artcle will focus on a building a `timeit!` macro, inspired by Python's [`timeit` module](https://docs.python.org/3.8/library/timeit.html). It will allow us to log the execution time for closures & functions; here's a usage example:

```rust
fn wait_for_it() -> String {
    std::thread::sleep(std::time::Duration::from_secs(2));
    return String::from("...Legendary!");
}

fn main() {
    eprintln!("This is going to be...");
    let res = timeit!(wait_for_it());
    eprintln!("{}", res);
}
```

#### **`output`**
```
This is going to be...
'wait_for_it' took 2002 ms
...Legendary!
```

### Implementing timeit!
The essence of the syntax `timeit!` is trying to create shorthand for is:

```rust
{
    let start_time = std::time::Instant::now();
    // Code to time goes here
    eprintln!("Took {:.3} ms", start_time.elapsed().as_millis());
}
```

## Wrapping a Closure via Matching and Expanding
The first use case has the most straight-forward since the closere is matched in `macro_rules!` as a single `expr` match type:

```rust
macro_rules! timeit {
    // This match captures the `expr` passed in as `$e`,
    // which the macro will assume is callable (E.g. a closure or function)
    ($e:expr) => {{
        // Before calling `$e`, track the current instant
        let _start = std::time::Instant::now();
        // `$e` could return something (or the implied unit struct `()`), so capture that in `_res`
        let _res = $e();
        // Log the elapsed time
        eprintln!("Took {:.3} ms", _start.elapsed().as_millis());
        // and return whatever our closure returned
        _res
    }};
}
```

Which allows us to use the macro like:

```rust
use std::io;
use std::fs::read_to_string;

fn main() -> io::Result<()> {
    let file_contents = read_to_string("path/to/file.txt")?;
    let result = timeit!(|| {
        my_lib::process_file(&file_contents)
    });
    println!("{}", results);
    Ok(())
}
```

#### **`output`**
```
Took 0.150 ms
Results: ...
```

## Wrapping a Function Call
Much like Rust's `match` blocks, ordering of rules is significant and the first pattern that matches will get executed. We can use this to our advantage and have a match rule that will attempt to match on function calls so that we can automatically include the function name in the logging output (to reduce ambiguity in code with multiple `timeit!` calls).

### Repetitive Matches and Expansion
Match rules can match an arbitrary number of homegenious items, much like the values passed into `vec![...]`. There isn't a match rule setup for each possibility of passed arguments, but rather the variadic arguments are captured in a match and the macro rule block can repeat syntax for each arg (like the `temp_vec.push(...);` lines in `vec![]`).

Given a macro call like:

```rust
vec![10, 20, 30]
```

The match rule could look like:
```rust
( $($args:expr),*) )
//     ^--- A repeating series of `expr` with non-captured comma separators
```

After capturing, a representation of the matches might look something like:
```
$args = [10, 20, 30]
```

When using the captured `$args` in the macro block, they can be re-assembled with comma separators like:

```rust
let mut temp_vec = Vec::new();
/*
 v-- things inside $( .. )* are repeated per item */
$(
   temp_vec.push($args);
)*
```

### Exploding and Re-assembling a Function Call
In order to attempt matching a function call, we'll place a new match rule at the beginning of our `macro_rules!` block, so when it doesn't match the rule the macro will proceed to the next rule to find a match:

```rust
macro_rules! timeit {
    /*  |--- function name (in this case: slow_sum)
        v     v--- the open paren before the function args   */
    ($n:ident ( $($args:expr),*)) => {{
    /*                ^          ^--- the closing paren after the function args)
                      |--- A repeating series of `arg` with non-captured comma separators */
        let _start = std::time::Instant::now();

        //              v--------v -> repeat each arg with following comma
        let _res = $n( $( $args, )* );
        //          ^--- our captured function name

        // Use the function name (ident) in the log
        eprintln!("'{}' took {:.3} ms", stringify!($n), _start.elapsed().as_millis());

        // return whatever the result of our function is (pass-through, like `dbg!()`)
        _res
    }};
    ($e:expr) => {{ .. }}  // <-- our existing rule
}
```

And we can see the effect of our macro via `cargo-expand`, targeted at our macro tests:

```rust
#[test]
fn test_simple() {
    timeit!(|| { std::thread::sleep(std::time::Duration::from_secs(1)) });
}
```

Becomes (comments added by me):

#### **`$ cargo expand --lib --tests`**
```rust
fn test_simple() {
//  v-- Start of our macro block
    {
        // This you should recognize
        let _start = std::time::Instant::now();
        // And this is `$e();` expanded
        let _res = (|| std::thread::sleep(std::time::Duration::from_secs(1)))();
        // Followed by the expansion of `eprintln!()`
        {
            ::std::io::_eprint(::core::fmt::Arguments::new_v1_formatted(
                &["Took ", " ms\n"], // <-- This is us!
                &match (&_start.elapsed().as_millis(),) {
                    (arg0,) => [::core::fmt::ArgumentV1::new(
                        arg0,
                        ::core::fmt::Display::fmt,
                    )],
                },
                &[::core::fmt::rt::v1::Argument {
                    position: 0usize,
                    format: ::core::fmt::rt::v1::FormatSpec {
                        fill: ' ',
                        align: ::core::fmt::rt::v1::Alignment::Unknown,
                        flags: 0u32,
                        precision: ::core::fmt::rt::v1::Count::Is(3usize),
                        width: ::core::fmt::rt::v1::Count::Implied,
                    },
                }],
            ));
        };
        _res
    };
}
```

And our function call example:
```rust
#[test]
fn test_ext_multiple_args() {
    fn slow_sum(a: u32, b: u32) -> u32 {
        std::thread::sleep(std::time::Duration::from_secs(2));
        a + b
    }
    let res = timeit!(slow_sum(5, 9));
    eprintln!("Slow sum result: {}", res);
}
```

Becomes (comments added by me):

#### **`$ cargo expand --lib --tests`**
```rust
fn test_ext_multiple_args() {
    fn slow_sum(a: u32, b: u32) -> u32 {
        std::thread::sleep(std::time::Duration::from_secs(2));
        a + b
    }
    //        v-- Start of our macro block
    let res = {
        let _start = std::time::Instant::now();
        let _res = slow_sum(5, 9);
        {
            ::std::io::_eprint(::core::fmt::Arguments::new_v1_formatted(
                &["\'", "\' took ", " ms\n"],
                //        v-- The function name, after `stringify!()`
                &match (&"slow_sum", &_start.elapsed().as_millis()) {
                    (arg0, arg1) => [
                        ::core::fmt::ArgumentV1::new(arg0, ::core::fmt::Display::fmt),
                        ::core::fmt::ArgumentV1::new(arg1, ::core::fmt::Display::fmt),
                    ],
                },
                //.. truncated for brevity
            ));
        };
        _res  // <-- returning the function result
    };  // <-- end of the macro block
    {
        ::std::io::_eprint(::core::fmt::Arguments::new_v1(
            &["Slow sum result: ", "\n"],
            &match (&res,) {
                (arg0,) => [::core::fmt::ArgumentV1::new(
                    arg0,
                    ::core::fmt::Display::fmt,
                )],
            },
        ));
    };
}
```

There we have it! Hopefully the macro syntax is a little more clear with this example. Check out [the full example](https://github.com/thepacketgeek/rust-macros-demo/blob/master/timeit/src/lib.rs) for even more details, along with some tests to show usage.