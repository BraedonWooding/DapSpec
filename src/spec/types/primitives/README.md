# Primitive Types

Primitive types are not only inbuilt into the language but also have some sort of relation to physical hardware (typically special instructions).

The following are all the current primitives that are defined in Dap.

- [Booleans] — Logical Statements
    - While these have no special instructions (other than arguably jump'like instructions) they are still a fundamental type to be able to have logical conditions and are included as a primitive for that reason.
- [Numerics] — Integers/Floating Points/Fixed Points
    - These have specially defined math operations and in some cases optimal instructions (for things like rqsrt/counting bits)
- [References] — References to other types (i.e. 'Pointers')
    - These have defined load/store instructions and act as an indirect memory location to some data
- [Functions] — Native Function References
    - Function references also have unique instructions depending on the scenario of call, typically some sort of instruction that jumps, saves the stack, and sets up the return address register.
    - This doesn't include [Closures] which aren't primitive and are required to have more functions that suspend.  Any function that returns a non-trivial job (i.e. asynchronous/multi return functions) typically require a closure and can't be represented by a primitive function reference.
- [Numerical Vectors] — Specialised data structures that map to special processor instruction sets such as AVX/SSE
    - Not to be confused with `std::vector` from the C++ standard library

[Numerics]: numerics.md
[Booleans]: booleans.md
[References]: references.md
[Functions]: functions.md
[Numerical Vectors]: numerical_vectors.md
