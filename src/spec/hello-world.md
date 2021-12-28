# Hello World

<div class="warning">

WIP

</div>

Hello World is a common introduction to a language, while I doubt it's teaching usefulness; it would be remiss to not include it, even if just purely for tradition.

```dap
use std::io::{Out};

main :: \() -> Out.lock.write("Hello World\n");
```

The parentheses there are optional too, and it's quite common (and more typical of the style) to just write it as follows:

```dap
use std::io::{Out};

main :: \() -> Out.lock.write "Hello World\n";
```

You can capture a lock very easily in Dap.

```dap
use std::io::{Out};

main :: \() -> {
    // this lock exists for the duration of the 'main' function
    out :: Out.lock;
    
    // you can separate it up however you want
    out.write "Hello";
    out.write " World";
    out.write "\n";
}
```

A more common/alternative syntax let's you capture variables for a short defined lexical scope.

```dap
use std::io::{Out};

main :: \() -> {
    with! out :: Out.lock -> {
        out.write "Hello";
        out.write " World\n";
    }
};
```

> `with!` is actually defined as a 'function-block' macro, these take in a `function-block` (defined as `(...args) -> {}`) that allows for you to shorten it as just `with! out :: Out.lock -> out.write "Hello World\n";`

## Too many 'ways to do something'?

There are often a lot of alternatives in Dap for syntax, however, there is only one 'preferred' syntax.

For example, in the Hello World example any linter would recommend:
- Never the manual solution of `{ out :: Out.lock; ... }`
- If it's a single capture and a single statement it would prefer `Out.lock.write "Hello World\n"` (i.e. if you have 3 in a row it would recommend a block syntax)
- If there are multiple declarations in a row OR there are multiple statements, then `with! out :: Out.lock, err :: Err.lock, ... -> { ... }` is preferred.
