# Booleans

```
view bool for Primitive, Comparable, LogicalOperators {
    bitwise_or  @operator | for \(other: Self)[self: Self]: Self;
    bitwise_and @operator & for \(other: Self)[self: Self]: Self;
    bitwise_xor @operator ^ for \(other: Self)[self: Self]: Self;
}
```

The boolean type `bool` has two defined values `true` and `false`.  Both are not unique keywords but are instead global values provided by the compiler.  The boolean type implements the following views; [Comparable], [Primitive], [LogicalOperators] and implementation operator overloads for the bitwise operators; `|`, `&`, `^` (since that `~` is not implemented the [BitwiseOperators] constraint isn't implemented).

These global values exist in the `std:core:internal` package, which is implemented entirely internally by the compiler and can't be replicated in userspace.

The boolean type has a size of `1 byte` and an alignment of `1 byte`, the `false` value has an internal bit pattern of `0x0` with the true value having a default internal bit pattern of `0x1`.

> It's [malformed behaviour] (in user-land) to construct a boolean such that it's value does not match either of the `true` or `false` bit patterns.  The conforming conversion `boolean(i: int)` converts any given integer to a boolean value and should always be used on any potentially non-conforming value (for example from C FFI).

The following operations are defined on booleans and have their usual logical definition;

- Logical NOT (`!`)
- Bitwise OR (`|`)
- Bitwise AND (`&`)
- Bitwise XOR (`^`)
    - Which can be simply defined as just `!(a & b)`
    
As well as the [lazy boolean operators] `&&` and `||`.

There is also the following comparison operations `==`, `!=`, `>=`, `>`, `<=`, `<` which are defined as per their usual meaning.

TODO: Truth tables would be nice since this is meant to be a specification.

The compiler is allowed to introduce booleans that have internal bit patterns that don't conform to the above standards in the following cases (for both `true` and `false`), this doesn't break the malformed expression stated above since it occurs in compiler-land;

- In an expression that evaluates to a conditional or used solely in a conditional, the result may manipulate the definition such that it can use more optimal instructions.
    - For example, rather than coercing `x + 10 > 20` to a boolean result it may simply just perform a jump condition on the value `x - 10 > 0`.
    - Keep in mind that this is a mostly trivial rule, and something that the vast majority of optimisers utilise (LLVM for example), the benefit is mainly in defining this said optimising behaviour.
    - This does actually mean that the bitwise not operator is potentially equivalent to the logical not operator, granted that it would define `0` as false, and `-1`/(the maximum value for that word) as true.  In some cases, this could provide more optimal code that avoids multiple jumps (in cases where something like a `SETZ/XOR` aren't suitable).

[malformed behaviour]: /spec/behaviour.md#malformed-behaviour
[lazy boolean operators]: /spec/syntax/operators.md#lazy-boolean-operators
