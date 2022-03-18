# Functions

<div class="warning">

WIP: Needs rewriting

</div>

Dap functions consist of a series of parameter definitions followed by a return value, and then by a evaluable block.  A simple definition is as follows;

```dap
answer_to_life_the_universe_and_everything :: \(): int _32 -> 42;

// Any block is valid here.
add :: \(a: int _32, b: int _32): int _32 -> {
    // last value doesn't have a `;` to signify return.
    a + b
};
```

## Function Parameters and Return Type

Function parameters follow the typical rules for variable declarations and can have complex type definitions.  They can also have tags applied to either the type or the parameter name.

```dap
divide :: \(a: f32.finite, b: { x: f32.finite | x != 0 }): f32.finite @Num.Flt.InPrecise(moreThan: {a, b})
-> a / b;
```

In the case you have multiple return types you can return them through a tuple-like.

```dap
// You can use the short hand form too for parameter types
div_and_rem :: \(a: int _32, b: { int _32 | _ != 0}): (int _32, int _32)
-> (a / b, a % b);
```

If you name the return value, you don't need an explicit final value.

```dap
div_and_rem :: \(a: int _32, b: { int _32 | _ != 0}): (div: int _32, rem: int _32)
-> {
    div :: a / b;
    rem :: a % b;
    // given that every value in the return value is specified by all exits
    // of this function, then you don't need an explicit return.
}
```

To avoid mistakes with returns you always have to specify the return type, the only exception is in the case when the function type has already been defined for example when you are passing the function to another function or have specified the type in the variable.  For example;

```
div_and_rem for \(a: int _32, b: { int _32 | _ != 0 }): (div: int _32, rem: int _32)
    :: \(a, b) -> (div: a / b, rem: a % b);

// and in a similar way if it's a single parameter with a known type
// you can elide the parentheses
myArray.map(\x -> x * x);
```

## Function Body

As with other blocks, the last expression is the result of that block, and in this case the resolved value of the function.  The examples above indicate the vast majority of how function bodies are formed.

Dap doesn't contain a 'return' keyword, this is primarily because errors are handled as transactions making propagation implicit.  For example the following code in Rust doesn't need an early return value due to propagation of errors.

```rust
use std::fs::File;
use std::io::prelude::*;
use std::io;
use std::fs::OpenOptions;

struct Info<'a> {
    name: &'a str,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    let mut file = OpenOptions::new().create(true).append(true).open("my_best_friends.txt")?;
    // Early return on error
    file.write_all(format!("name: {}\n", info.name).as_bytes())?;
    file.write_all(format!("age: {}\n", info.age).as_bytes())?;
    file.write_all(format!("rating: {}\n", info.rating).as_bytes())?;
    Ok(())
}

println!("{:?}", write_info(&Info { name: "Chippie", age: 6, rating: 100 }));
println!("{:?}", write_info(&Info { name: "Amelia", age: 11, rating: 100 }));
let mut buffer = String::new();
let mut file = File::open("my_best_friends.txt").unwrap();
file.read_to_string(&mut buffer);
println!("{}", buffer);
```

And writing the above example (taken from Rust documentation) in a Dap like way (maintaining similarity as much as possible).

```
use std::io::{File, Out};

data Info {
    name: String;
    age: int _32;
    rating: int _32;
}

write_info :: \(info: Info): Job -> {
    with file :: File.open("my_best_friends.txt", .CREATE | .APPEND) -> try! {
        // you would probably just do this in a single call... but you could do that
        // in the rust version too, so let's just keep it consistent.
        file.write_all $"name: {info.name}\n";
        file.write_all $"age: {info.age}\n";
        file.write_all $"rating: {info.rating}\n";
    };
}

with out := Out.lock() -> {
    try! write_info { name: "Chippie", age: 6, rating: 100 } else out.write($"{_}\n");
    try! write_info { name: "Amelia", age: 11, rating: 100 } else out.write($"{_}\n");

    with file :: File.open("my_best_friends.txt", .READ) unwrap {
        contents :: file.read_all;
        out.write($"{contents}\n");
    };
}
```

The major difference is the use of the `try` macro as opposed to the `?` symbol, this is explained in more detail in the Jobs section of the documentation.  There are a few other key important differences here, which will be recapped here just for completeness but you should refer to the relevant section for more details;

- Parentheses to call functions is optional and are typically elided in the case you have a single argument.  In some cases (such as when you have multiple try statements in a row like `try (x) else try (y) ...`) it can be more readable to have parentheses.
- Functions are implicitly asynchronous in the Dap example.  In the Rust example you would have to use the async equivalents (which don't exist in the standard library and instead require the use of libraries like tokio, or the use of a thread-pool).
- As long as you have attached a try at some point in the lexical structure of a function it applies implicitly to all chained calls, for example applying `try!` to a block, implicitly handles all errors possibly occurring in that block and applying it to a single function applies to all chain calls in that statement i.e. `try! (foo.X.Y.Z) else out.write($"{_}\n")` will support errors being raised in either X, Y, or Z.
- `unwrap` is simply just a function that is `unwrap :: \(x: Job[$T]) T -> try (x) else fatal($"Unwrap Failed: {_}\n")`
  - `$T` refers to a generic which will be discussed in more detail later in this documentation.
- `try` has a more complicated definition and is a macro, the main reason why is due to the ability to chain multiple tries i.e. `try! (x) else try! (y) else try! (z) else out.write($"{_}\n")`.
  - The definition is explained in the macros section but in the case of `try! (x) else out.write($"{_}\n")` roughly translates to `with (res :: x) if (res.isOk) res.value else out.write($"{res.error}\n")`

## Is everything a function?

Initially, it might appear as though there isn't a distinction between a function and a primative value for example;

```
prim :: 2;
prim_fn :: \() -> 2;
prim_resolved :: prim_fn;

assert! prim == prim_resolved;
```

This is partly due to the fact that parentheses for function calls are implicit but as a deeper reason, firstly Dap defines expression as 'resolvable', which is defined as follows

> A resolvable expression eventually evaluates to a given result.

There are many sub-types of this, such as;
- Indeterminstic/Deterministic resolvable expressions: given a set of input values; it'll produce the same/a potentially different result.
- Indeterminate/Determinate duration resolvable expressions: the period of time in which it'll resolve is unknown/known to some degree.
  - Typically, this is more of a scale then a binary definition.
  - As a sub-type, there is infinite duration functions (non-terminating).

Thus, it's not so much that everything is a function but rather everything resolves to a similar 'result' like definition.  What typically confuses people about this sort of definition in functional programming (and also in Dap for a similar reason) is partial evaluation.  For example, the following function would cause a crash if ever evaluated (integer division by 0) but since it's never evaluated it's fine.

```
// _ is a special 'consume' variable for when you don't want to retain the result
// we can't do something like `foo :: 1 / 0` here, because that would be retaining the result (causing almost certain evaluation)
// inside the variable foo, which is then up to the compiler to decide if it wants to evaluate the result or not.
// (Typically in cases like this it would evaluate a variable in debug but not release given the variable will probably be optimised away in release).
// however, of course; in this case given that nothing is retaining the value the compiler isn't likely to force evaluation of the expression.
_ :: 1 / 0;
```

Unlike functional languages like Haskell, you have significantly less control over when expressions get evaluated.  Even in the above case, a compiler could decide to present an error (at compile time or run time), it's just something that is very likely to be removed (given that it's so simple).

Typically, this makes no difference to the way you'll approach problems in Dap.  The major difference, however is with 'non-terminating resovable expressions' (i.e. infinite loops).  We'll avoid talking about the simplest case of non-termination, that is something that exits through a fatal or an exit (counts as non-termination since it's non-typical termination since the function never evaluates) since that's a trivial case to account for (we have the tag `@NoReturn` for this exact case).

A classical infinite loop is a generator function, for example a counting iterator for all natural numbers (n > 0 given n is an int _32).

```
natural_nums :: \(): Job Coroutine[int _32] -> {
    n :: 1;
    loop! {
        yield! n++;
    }
}
```

This is may be a bit more obvious than an infinite Haskell list as defined given the Job/Couroutine return value but just to ensure it's clear:
- This is resolvable to a value (after a reasonably determinate period of time at that), it's just that it's resolvable to an infinite amount of values.
- In this way, it only actually becomes a non-terminating resolvable expression if it's evaluated infinitely, in that way it's up to the caller to whether or not this terminates.
- Thus, in all defined programs (granted that programs are defined as to eventually halt/resolve) it terminates after a finite number of iterations.

This sort of propagation of termination is the foundation behind the Job system, and is pretty much how we can make such clear decisions about programs.  In this way however, we presume that all user programs will actually terminate *eventually*.

For example, a game loop could look a little like below:

```
// tail call recursive call
game_loop :: \() -> {
    input :: process_input();
    end_game :: perform_update(input);
    render_frame();

    if (!end_game) game_loop();
}

// will be literally identical to below (or at-least behaviourally identical)... but you can write it either way
game_loop :: \() -> {
    end_game := false;

    while !end_game {
        input :: process_input();
        end_game = perform_update(input);
        render_frame();
    }
}
```

Even if it's less explicit and ending the game is done through a `@NoReturn` function call (not typically recommended since it's nicer to actually clean up resources and flush files and so on), that's actually fine too, because presumably that would be performed in perform_update, which means that perform_update therefore inherits (to some degree) the ability to have a `@NoReturn` which thus implies that `game_loop` can terminate.
