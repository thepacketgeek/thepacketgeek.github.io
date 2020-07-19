+++
title = "Improved Retryable Logic with Macro Setup"
date = 2020-08-12
author = "Mat"
in_search_index = true

[taxonomies]
tags = ["rust"]
+++

We've now [built a simple `retry!` macro](@/rust/macros/macro-matching-and-nesting.md) where the *limited* logic is contained in the macro_rules. This article will focus on an improved `Retryable` struct (with `RetryStrategy`) that we'll then build a new macro to simplify instantation for.

## Retryable & RetryStrategy
Forgoing macros for a bit, let's setup some retry structs and implementations. First is a `Retryable` struct to contain our function/closure to retry, and a `RetryStrategy` with options for retrying (number of retries, delay, etc.):

<!-- more -->
```rust
pub struct Retryable<F, T, E>
where
    F: FnMut() -> Result<T, E>,
{
    inner: F,
    strategy: RetryStrategy,
}

/// Specification for how the retryable should behave
pub struct RetryStrategy {
    retries: usize,
    delay: RetryDelay,
}

pub enum RetryDelay {
    Fixed(std::time::Duration),
    // TODO: More options here
}
```

The core of our implementation for this struct looks like the retry logic from `retry!`, although we now use the delay options from `RetryStrategy`.

```rust
impl<F, T, E> Retryable<F, T, E>
where
    F: FnMut() -> Result<T, E>,
{
    /// Start calling the wrapped function, responding to Errors
    /// as the specified strategy dictates
    pub fn try_call(&mut self) -> Result<T, E> {
        let mut retries = self.strategy.retries;
        let mut delay_time = Duration::from_millis(0);
        loop {
            std::thread::sleep(delay_time);
            let res = (self.inner)();
            if res.is_ok() {
                break res;
            }
            if retries > 0 {
                retries -= 1;
                delay_time = self.next_run_time();
                continue;
            }
            break res;
        }
    }

    fn next_run_time(&self) -> Duration {
        match self.strategy.delay {
            RetryDelay::Fixed(delay) => delay,
        }
    }
}
```

Breaking out this logic into the `RetryStrategy` gives us much more flexibility with retrying, but now we have a problem with a more tedius setup:

```rust
let strategy = RetryStrategy::default().with_retries(3).to_owned();
let mut r = Retryable::new(succeed_after!(2), strategy);
let res = r.try_call();
assert!(res.is_ok());
```

## Automating Retryable Setup
Luckily for us we have an awesome tool in the toolbox that we can use to make this setup much easier: a macro! Using some similar matching rules we used with `retry!`, we can setup a very flexible macro to allow for optional specification of retries:

```rust
macro_rules! retryable {
    // Take a closure with retry count
    // ```ignore
    // retryable!(|| { do_something(1, 2, 3, 4) }; retries=2);
    // ```
    ($f:expr; retries=$r:expr) => {{
        let _strategy = RetryStrategy::default().with_retries($r).to_owned();
        let mut _r = Retryable::new($f, _strategy);
        _r.try_call()
    }};
    // Take a function ptr, variadic args, and retry count
    // ```ignore
    // retryable!(my_fallible_func, 0, "something"; retries=5);
    // ```
    ($( $args:expr $(,)? )+; retries=$r:expr) => {{
        retryable!(|| { _wrapper!($($args,)*)}; retries=$r)
    }};
```

We now have a very similar usage to our previous `retry!` macro:

```rust
let res = retryable!(sometimes_fail, 10; retries = 15);
assert!(res.is_ok());
```

Although how about setting the `delay` value via macro invocation? For that we'll need to add a couple more rules to support `delay` on its own, and also with both `retries` and `delay` specified:

```rust
macro_rules! retryable {
    // ... existing rules

    // Take a function ptr, variadic args, and delay time (seconds)
    // ```ignore
    // retryable!(my_fallible_func, 0, "something"; delay=5);
    // ```
    ($($args:expr$(,)?)+; delay=$d:expr) => {{
        retryable!(|| { _wrapper!($($args,)*)}; delay=$d)
    }};
    // Take a function ptr, variadic args, retry count, and delay time (seconds)
    // ```ignore
    // retryable!(my_fallible_func, 0, "something"; retries=2; delay=5);
    // ```
    ($($args:expr$(,)?)+; retries=$r:expr; delay=$d:expr) => {{
        retryable!(|| { _wrapper!($($args,)*)}; retries=$r; delay=$d)
    }};
```

Now look at how easy `retryable!` is to use!

```rust
let res = retryable!(sometimes_fail, 10; delay = 2);
assert!(res.is_ok());

let res = retryable!(|| {sometimes_fail(10)}; retries = 15; delay = 1);
assert!(res.is_ok());
```

Much better that the manual `Retryable`/`RetryStrategy` setup!! Check out the [full implementation](https://github.com/thepacketgeek/rust-macros-demo/blob/master/retryable/src/lib.rs#L174) to see how the macro also supports `delay` timers and more.