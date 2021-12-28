# Lexical Structure

<div class="warning">

WIP

</div>

Dap programs should be stored in the UTF-8 (Unicode) format.  This format should be stored *without* the use of a DOM block.  If an invalid format is detected the compiler should exit and present an error.

## Keywords

Dap has not finalised it's list of keywords and prior to that occurring any specification update may include new keywords, this is not extremely likely to break existing code, but it could (I suppose).

The following keywords exist and can't be used at all in code, these will be referred to as "strict".

> **<sup>Strict Keywords:<sup>**\
> KW_BREAK: `break`\
> KW_CONTINUE: `continue`\
> KW_FOR: `for`\
> KW_IF: `if`\
> KW_ELSE: `else`\
> KW_WHILE: `while`\
> KW_RECORD: `record`\
> KW_UNION: `union`\
> KW_VIEW: `view`\
> KW_PACKAGE: `package`\
> KW_MATCH: `match`

You will notice that very few keywords exist in Dap (currently 10) as compared to other languages (such as Rust (~58), C# (~79), Zig (~49), and so on, even C (at 32) and C++ (at 60) have more keywords).  This is because Dap at it's core is very simple as a language, and relies instead on macros to implement most of it's 'auxiliary keywords' (i.e. useful language features such as try, in a similar way to Rust).

> If you ignore the type keywords (and auto) in C, there are still 20 keywords (from my counting).  To my knowledge this does put Dap as one of the languages (non-esoteric) with the fewest keywords that doesn't replace them with symbols.

It's very unlikely for Dap to introduce new keywords in the future.  The simplicity of the core language is one of the reasons why I really like Dap in it's current 'specification state'.
