# References

To facilitate the sharing of objects references can be created, references refer to the location of the object and are themselves [value types], meaning they have copy semantics.  Unlike most high level languages that don't have a garbage collector, Dap doesn't provide you the ability to define arbitrary pointers.

References have multiple implementations including:

- Reference Counted References
- Scope/Lexical References
- Single Owner References
- and others...

The simplest is the unsafe raw reference which is analogous to a reference in C.  To capture a reference it's as simple as follows:

```
// define a variable, we have to actually manually size this
// because otherwise it won't be able to resolve the type (which is required
// when references are involved).
x for int:32 := 2;
y for int:32 := 9;

// our reference initially is going to point to x
// for this to actually work we have to concrete 'x' to an actual
// memory address, since it could actually just be a register.
ref for Ref:Scoped[int:32] := @addressOf x;

// Once you've captured a reference it behaves exactly as if it was 'x'
// it's just sharing the memory location of 'x'.
Out.lock().write($"ref: {ref}, x: {x}, y: {y}\n"); // writes '2', '2', '9'

// We can mutate x
x = 4;
Out.lock().write($"ref: {ref}, x: {x}, y: {y}\n"); // writes '4', '4', '9'

// We can mutate ref
ref = 8;
Out.lock().write($"ref: {ref}, x: {x}, y: {y}\n"); // writes '8', '8', '9'

// We can change the reference by redeclaring the variable
// redeclarations are only valid for some types (references being one of them)
// and since it has to maintain the same type we don't have to restate it.
ref := @addressOf y;
ref = 2;

Out.lock().write($"ref: {ref}, x: {x}, y: {y}\n"); // writes '2', '8', '2'

// we can even redeclarate it as a constant
ref :: @addressOf y;

// in which case it's readonly and any attempt to write to 'ref' would
// raise a compiler error, but we can write to 'y' still
y = 6;

Out.lock().write($"ref: {ref}, x: {x}, y: {y}\n"); // writes '6', '8', '6'

// finally we can even point 'ref' to something else
// this kinda both creates a reference to a reference and no reference at all
// because it'll behave as though it's just 'y' (since ref is pointing to y)
// but it'll adjust as 'ref' changes.

// we have to make this immutable though since 'ref' is immutable
other for Ref:Scoped[Ref:Scoped[int:32]] :: @addressOf ref;
other := @addressOf y; // but we can make a mutable ref to y.

// writes '6', '8', '6', '6'
Out.lock().write($"ref: {ref}, x: {x}, y: {y}, other: {other}\n");

// change 'ref' to point to 'x', which thus indirectly causes 'other' to
// point also to 'x'
ref := x;
// writes '8', '8', '6', '8'
Out.lock().write($"ref: {ref}, x: {x}, y: {y}, other: {other}\n");
```

The reference view constraint is as follows:

```
// T must be a statically sized type or atleast known if dynamic
// (for example you can bind a reference to a member OF a dynamically sized
// array if the member has a known size and it's homogeneous).
// this is the simple case where T is not a recursive ref and we can just evaluate
// it directly to a known value.
view Ref[T for (KnownSize, T != Ref)] {
    // the '.' operator is a special operator which not only refers
    // to member access but also to any operation that requires evaluation
    // of the underlying value i.e. math/printing/whatever
    deref @operator . for \(on: Self)[]: T;
}

// we can use recursive type generation definitions to generate a unique
// type signature for each indirection layer to simplify constraints
// the way this is done is semi-similar to a linked list.
// we just define the reference recursively knowing that eventually
// it must do 'for Ref[T]' where T is a known size and non ref (or reach
// a pointer where it's no longer a ref but is dynamic and we can do an error)
view Ref[K for (K == Ref[T])] for Ref[T] {}

// Some references define one of two types of specialised references
// which are specified below...

view RefIndexForwards[T for KnownSize] for Ref[T] {
    // used like ref:2 to dereference can use expressions of course
    // i.e. ref:(n - 1), uptr is a special type that has a size which
    // is implementation defined but is guaranteed to equal the word size
    // this does mean you can't index backwards with a pointer ref.
    // unsigned type 
    index @operator : for \(other: int:uptr:pos)[self: Self]: T;
}

view RefIndexBackwards[T for KnownSize] for Ref[T] {
    // used like ref:-2 to dereference can use expressions of course
    // i.e. ref:(0 - n), unsigned types can be taken as either negative or
    // positive because well they are 'unsigned'.  So it's saying that
    // basically the input number will be negative and should be treated as such
    // even though it will be stored identically to a positive number.
    index @operator : for \(other: int:uptr:neg)[self: Self]: T;
}
```

An example implementation (of just the basic deref) is given below for Ref:Raw

```
// simplifies implementation since we don't have to copy it twice
// abstract views are quite useful in this way.
record Ref:Raw[T] {
    raw for int:uptr:pos;
}

record Ref:Raw:Base[T for (KnownSize, T != Ref)] for Ref[T], Ref:Raw {
    // is a simple implementation in the simple case of no recursion
    deref :: \()[self, T for (KnownSize, T != Ref)]: Ref:Raw[T]
         -> @unsafe.volatile.load[T] raw;
}

record Ref:Raw:Rec[K for (K == Ref[T])] for Ref[T], Ref:Raw {
    // in this case we have recursion (since Ref[K] could recurse and could
    // even be other types of references), we rely on the 'for Ref[T]' to
    // recurse all the way down using the return types.
    // we still have to implement the deref recursively though, the hardest thing
    // here is to actually know what the 'return' type of 'deref' is here
    // it's actually very hard to find that out due to the recursive definition
    // and most likely would require self reflection i.e.
    // `@Type[Self].deref.return_type` or something similar.
    deref :: \()[self]
        -> deref >> @unsafe.volatile.load[K] raw;
}
```

This seems complicated but the key idea is that we are matching a type definition to the actual representation, this sort of recursion will then result in pretty optimal code, just a series of recursive-like loads (very inlinable).

WIP: Move to generics...

The aspect of this that makes it different to how templates work (and not have the same issues) is due to the fact that the recursion happens at a type level not at an implementation/expansion level, this means that when we are type resolving; we can fully resolve the type of our recursive generics without the need of evaluating any real code.  In this case it's actually a very efficient evaluation since we are just mapping an already known type to the underlying raw structure i.e. we know the type is `Ref[Ref[Ref[Ref[int:32]]]]` and we just have to figure out the actual record types it maps to, this is mostly just some simple constraint checks (it's a little tricky to do this in the event of placeholder types, but still not particularly difficult).

This means that while the language is turing complete (I mean I guess it is...) it's only able to 'describe' a turing complete language not necessarily evaluate to a turing complete result at type evaluation.  That would require the use of comptime evaluated functions (which is fine and doesn't lead to the nightmare problems of templates).

[value types]: /spec/types/common_types_and_views.md#ValueType
