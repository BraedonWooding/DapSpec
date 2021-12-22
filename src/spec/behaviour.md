# Unspecified/Undefined/Malformed Behaviour

Outside of defined behaviour, there are 3 formal definitions of behaviour in Dap.

## Unspecified Behaviour

Unspecified behaviour simply defers the specification to another specification, for example in C signed integer overflow/underflow is unspecified (and left up to the processor/system).

At this current point, no core behaviour is unspecified.  Some behaviour, however is unspecified in some places in the language;

- The use of any compiler plugins (unofficial ones) that extend the compiler with additional features are implicitly unspecified by the specification, but typically would have stable APIs.
- Vectorisation extensions (SSE/AVX/...) TODO(Update): things like alignment rules/instructions can differ processor by processor, this would require research to say how much of it is unspecified.

<div class="warning">

WIP: There is surely more unspecified behaviour in the language.

</div>

## Undefined Behaviour

Undefined behaviour typically has a definition where simply the compiler makes no guarantees on what will occur, traditionally undefined behaviour silently causes issues or crashes (i.e. segmentation/page faults due to null dereferences).

Dap contains very little undefined behaviour, outside of the following;

- Programs that don't terminate are undefined, Dap presumes every program written will eventually terminate.
    - While Dap cannot guarantee behaviour, typically the expectation is simply that the program will compile (if no other issues persist) but not halt, and just process forever.
- Certain FFI calls/behaviour is undefined due to the nature that is making FFI calls
    - This includes cases such as; ABI compatability (i.e. the types you defined in the FFI signature are the actual types expected), the FFI call will eventually terminate, and all the behaviour invoked by the foreign language is well defined by the language's specifications/references.

Any other undefined behaviour (there may be a few points not mentioned here) is treated as a bug in the compiler.

## Malformed Behaviour

Malformed behaviour (to my knowledge) is a new concept in Dap that is roughly analogous to the intentions behind undefined behaviour in other languages.  Malformed behaviour however causes a panic ('crash') to occur in the event of detection and for that reason it's well defined/specified in the specification, what makes it different is the fact that it can go unchecked/undetected in certain cases, such as:

- Compiler generation bugs (i.e. where the compiler produces incorrect and non-conforming code)
- [Unsafe] (`@unsafe`) code, where the user explicitly states that a block of code can ignore a certain subset of 'safe' Dap, including operations such as; dereferencing raw references, defining unsafe ownership relations, and more.

[Unsafe]: unsafety
