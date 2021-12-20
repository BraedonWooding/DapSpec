# Notation

## Grammar

The following notations are used by the *Lexer* and *Syntax* grammar snippets:

| Notation          | Examples                              | Meaning                                   |
|-------------------|---------------------------------------|-------------------------------------------|
| CAPITAL           | KW_IF, LIT_INT                        | A token produced by the lexer             |
| _ItalicCamelCase_ | _ForStatement_, _Type_                | A syntactical production                  |
| `string`          | `x`, `while`, `*`                     | The exact character/s                     |
| \\x               | \\n, \\r, \\t, \\0, \\0x1F4A9         | The character represented by this escape  |
| ( )               | (`,` _Parameter_)<sup>?</sup>         | Groups items                              |
| x<sup>?</sup>     | (`:` _Type_) <sup>?</sup>             | An optional item                          |
| x<sup>\*</sup>    | `(`(_Id_ `:` _Type_)<sup>\*</sup>`)`  | 0 or more of x                            |
| x<sup>+</sup>     | _Statement_<sup>+</sup>               | 1 or more of x                            |
| x<sup>a..b</sup>  | FRAG_HEXDIGIT<sup>1..6</sup>          | a to b repetitions of x                   |
| \|                | `u8` \| `u16`, _Block_ \| _Statement_ | Either one or another                     |
| \[ ]              | \[`b` `B`]                            | Any of the characters listed              |
| \[ - ]            | \[`a`-`z`]                            | Any of the characters in the range        |
| ^\[ ]             | ^\[`b` `B`]                           | Any characters, except those listed       |
| ^`string`         | ^`\n`, ^`*/`                          | Any characters, except this sequence      |
