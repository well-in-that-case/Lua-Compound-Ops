# Lua 5.4.4
## Compound Operator Support
### New Compound Operators:
- Modulo: `%=`
- Addition: `+=`
- Exponent: `^=`
- Bitwise OR: `|=`
- Subtraction: `-=`
- Bitshift left: `<<=`
- Bitwise AND: `&=`
- Float division: `/=`
- Bitshift right: `>>=`
- Multiplication: `*=`
- Integer division: `//=`

### Lua Compatibility
The bytecode produced by these operators is identical to their manual counter-parts (i.e, `a = a + 1`).

### Implementation Detail
- This implementation does not require additional reserved symbols.
- This simply alters the lexer to translate every compound operation into an assignment.
- Every time this happens, the lexer stores the binary operator token used in the compound assignment.
- The parser `restassign` function picks up the assignment because it was translated.
- The `restassign` function is modified to check for the saved lexer state token prior to an assignment.
- If a saved lexer state token exists (>0), then the `compound_assignment` function performs the operation.
- Afterwards, the `restassign` function resets the lexer state token (`ls->lasttoken`) for the next operation.

This is remarkably simple, taking ~1-3kb of code depending on how you optimize, expand, or secure it.
There are no new symbols, opcodes, metamethods. There's no problems with complex lvalue expressions.

### To-Do
This does not support tuple assignment yet.

```
This is Lua 5.4.4, released on 13 Jan 2022.

For installation instructions, license details, and
further information about Lua, see doc/readme.html.
```