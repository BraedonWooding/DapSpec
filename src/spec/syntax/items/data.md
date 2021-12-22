# Data (Struct/Enums/Unions)

Data is a general type in Dap that lets use specify multiple other types within it.

The simplest form is it's `struct` form, where it represents a (potentially) hetrogeneous product of types, each of them having a name that together forms what is referred to as a field.

```
data Point2D {
    x: int _32;
    y: int _32;
}
```

## Memory Layout

The memory layout can be specified through a tag, `@Representation(padding?: int'bytes, alignment?: int'bytes, layout: Layout)`.  Memory layout can be specified on fields as well as the `data` object itself, for example the the following;

```
data Example @Representation(padding: 4'bytes, alignment: 16'bytes, layout: .RAW_ORDER) {
    a: int'32 @Representation(padding: 1'byte);
    // this puts the padding at 4 bytes (min of 4 in data and of 1 for a)
    // 'a' is aligned to 4 bytes though not 2
    
    // the 4 bytes of padding would actually make this 8 bytes aligned
    // (all that is required for 64 bits), but given alignment is set to 16 bytes
    // this would add another 8 bytes of padding.
    b: int'64 @Representation(padding: 5'bytes)
    // padding here will be 5 bytes + 11 bytes to ensure alignment (thus 16 bytes).
}
```

`padding` refers to minimum spacing/offset (in bytes) between each object, by default this has a value of 0 but can be set to any positive number (including 0).

`alignment` refers to the required minimum alignment (in bytes) of each field, and will insert extra padding at the end of each field object to guarantee alignment.  This can be any positive number (including 0) and it's default value is based upon the layout.

`layout` can be any combination of the following (as bitflags).

- Ordering of fields (0x1-3) (only allowed on `data` definitions and any fields that have types which are themselves `data` definitions)
    - `.RAW_ORDER     = 0x1` the order of the fields is exactly as specified
    - `.OPTIMAL_ORDER = 0x2` the order of the fields is packed as to ensure the lowest padding (typically in descending order of size)
    - `.RANDOM_ORDER  = 0x3` the order of the fields is randomly generated, mainly intended for fuzzing the compiler/tooling (refer to Fuzzing documentation for more details).
- Special flags
    - `.TRANSPARENT   = 0x4` this field
- Miscellaneous
    - `.DAP     = .PACKED`
    - `.DEFAULT = 0x0` the default ordering is used (which is just `.DAP`)

- `@Representation(.STRICT)` 
- `@Representation(.NONE)` 
