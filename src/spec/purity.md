# Purity

'Purity' is a concept in computer programming, in which a pure function exhibits the following properties;

1. Provided identical inputs and context (i.e. globals typically) a function will provide an identical output.
2. No side-effects are exhibited by the function i.e. mutation of globals, I/O, ...

Typically, in languages a 'pure' function can refer to just a function that exhibits the second behaviour above.  Dap differentiates between the degree of purity via an enum.

```
@enum @flags FunctionPurity {
    UNKNOWN         = 0x0;  // default: get compiler to calculate purity
    DETERMINISTIC   = 0x1;  // property 1.
    NO_SIDE_EFFECTS = 0x2;  // property 2.
    IMPURE          = 0x4;  // explicitly state (or compiler calculates) function as impure.
}
```

> Deterministic does not infer non-chaotic, many cryptographic functions are chaotic (mathematical definition of chaos being that they are very sensitive to starting conditions and are very hard to predict) but provided the same input will encrypt to the same result and thus exhibit deterministic behaviour (providing identical output given identical inputs).

It's reasonably expensive to detect the purity of a given function, and typically it's a useful documenting feature, so it's typically recommended to tag all exported functions in a library with an appropriate purity flag via the `@purity(flags: FunctionPurity)` tag.

> It's currently an error for any standard library function to not get tagged.

> Purity flags do effect ABIs, so in the case you expect them to change in the future it's preferred you don't explicitly state them or tag them with impurity, even if at that moment they are pure.

> Functions that don't have an explicit (non-zero/non-unknown) purity and are actually pure will only use the pure property for optimistation when statically linking (since static linking can ignore certain ABI restrictions).  For this reason, they don't form part of the ABI and won't dynamically link with non-explicitly stated pure functions as if they are pure.  The main reasoning is to reduce ABI breaks.
