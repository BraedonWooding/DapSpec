# Common Types and Views

## `ValueType`

ValueType is a compiler specified constraint which simply just means that any assignment operation performs a copy by default instead of a move.

Implicitly ValueTypes have the following extra constraints defined:

```
view ValueType
{}

view Clone for ValueType
{
    clone :: \()[self] -> self;
}
```

[Clone](#clone) for ValueType's has a default implementation that is trivial as shown above.

## `Clone`

```
view Clone
{
    clone for \()[self: Self]: Self;
}
```

## `Comparable`

```
view Comparable
{
    equals      @operator == for \(other: &Self)[self: Self]: bool;
    notEquals   @operator != for \(other: &Self)[self: Self]: bool
        :: bool.not >> Self.equals;
}
```

> `>>` is the function composition operator, it goes both ways so we could also define it as `Self.equals << bool.not`.

### Side note on default definition

We could define it as `(!) >> (==)` (explicit parentheses are required) to avoid the type clarifications to the point that we could even write an implementation of the view as follows (if defining the view we would be forced to define the type still):

```
notEquals :: (!) >> (==);
```

But this is frowned upon for style reasons, it's significantly less readable (especially for beginners).  Typically, you won't have to give a definition of `notEquals` since the default one provided by the view is probably correct in every case, exception would be where it's more efficient to compute some other way.
