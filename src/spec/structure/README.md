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
> KW_WHILE: `while`
