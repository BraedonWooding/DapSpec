# Operators

## Lazy Boolean Operators

The following lazy operators are defined on booleans;

- `||` and `&&`

These are short circuiting 'lazy' operators that don't evaluate non-pure expressions (i.e. expressions with side-effects) if the result is already known.

> [Purity is an important concept for the Dap compiler](/spec/purity.md)

For example, `false && foo()` will not evaluate `foo()` (provided it's non-pure) since the result will be false regardless.

> In some cases provided pure functions the compiler can simplify the lazy operator to just a bitwise equivalent operator.

> If both expressions are pure the compiler is allowed to reorder boolean conditions for example, `pure(x) == pure(y - 10) || x <= y * 10` will likely be re-ordered to just `x <= y * 10 || pure(x) == pure(y - 10)` granted that `pure` is likely to be more expensive.  This can make quite a difference with some boolean cheap evaluations for example, `allowsJump && (path :: generatePath(grid, start, end))` is more likely significantly cheaper than `(path :: generatePath(grid, start, end)) && allowsJump` in the event they can't jump.

## Assignment Operators/Declaring Variables

There are 3 basic operators and a few compound ones.

- `=` performs a typical assignment
- `:=` acts as a mutable variable declaration i.e. `a := 2`
- `::` acts as a immutable variable declaration i.e. `b :: 2`

When declaring variables you can specify an explicit type using `for` but assignment still follows the above operators i.e.

```
// below are both immutable
x1              :: 2;      // infer type as Numeric and based on usage will define as float/int
y1 for float:32 :: 2;      // force coercion to a floating point number (32 bits)

// below are both mutable with the same type semantics as above.
x2              := 2;
y2 for float:32 := 2;
```

These behave very similarily to defining a constraint view along with an explicit cast, i.e. they are pretty much identical to below;

```
// Create a constraint view to force coercion.
view Tmp for float:32 {}

x1 :: 2;
y1 :: @cast[Tmp](2);

x2 := 2;
y2 :: @cast[Tmp](2);
```

We could of course just cast the type to `float:32` but in more complex cases you can specify more generic constraints which aren't so easy to specify, for example using a generic type requires a temporary constraint view in some cases where further specialisation is required.  If it's trivial the compiler *can* simplify the cast.

```
// presuming T is a generic type parameter, with a passed in value `x: T`
// we need the comptime check (that runs after basic type information)
// so that we only generate the foo branch when required.
@comptime if @Type[T].implementsView @View[ValueType] {
    foo for T, ValueType :: x;
}
```

In cases like above we need the use of a temporary view constraint to construct the complex type definition.  Something like `view Temp[T] for ValueType {}` (the generic constraint on the type is required, since views are not localised and so don't have context of what type/view `T` actually is).  It will then simplify the code above to just:

```
@comptime if @Type[T].implementsView @Type[Temp[T]] {
    foo for Temp[T] :: x;
}
```

### Compound Assignment Operators

After a variable has already been declared, given that it's mutable you can utilise the normal binary arithmetic, bitwise, and boolean operators with a `=` appended to follow the following style; `a OP= b` is equivalent to `a = a OP b` i.e. `a += b` is `a = a + b` and `a ||= b` is `a = a || b`;

The following operators are defined as compound operators:

- `+=`, `-=`, `*=`, `%=`, `/=`
- `&=`, `|=`, `^=`
- `||=`, `&&=`
