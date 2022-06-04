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
The bytecode produced by these operators is almost identical to their manual counter-parts (i.e, `a = a + 1`).
Compound assignment operators are typically faster with complex lvalue expressions. Conversly, simple lvalue expressions — for example, a simple local — are slower.

Simple lvalue expressions produce an extra MOVE opcode because this fork performs inside a temporary register, then moves the value into your object. Conversly, Lua optimizes manual expressions (`a = a + 5`) to directly modify the value. Complex lvalue expressions require less lookups because the temporary register will remember the value & share it, instead of requesting it twice.

Take this example code:
```lua
local a = { b = { c = { d = { e = { f = { value = 5 } } } } } }
local start = os.clock()
for i = 1, 10000000 do
    a.b.c.d.e.f.value = a.b.c.d.e.f.value + 5
end
print(os.clock() - start)
```
This will take ~1400ms to process. Whereas, if we replace it with this:
```lua
local a = { b = { c = { d = { e = { f = { value = 5 } } } } } }
local start = os.clock()
for i = 1, 10000000 do
    a.b.c.d.e.f.value += 5
end
print(os.clock() - start)
```
It takes only ~850ms to process. Compound operators offer a 48.8% performance increase with complex lvalue expressions.

### Implementation Detail
- This implementation does not require additional reserved symbols.
- This simply alters the lexer to translate every compound operation into an assignment.
- Every time this happens, the lexer stores the binary operator token used in the compound assignment.
- The parser `restassign` function picks up the assignment because it was translated.
- The `restassign` function is modified to check for the saved lexer state token prior to an assignment.
- If a saved lexer state token exists (>0), then the `compound_assignment` function performs the operation.
- Afterwards, the `restassign` function resets the lexer state token (`ls->lasttoken`) for the next operation.

This is remarkably simple, taking 3-4kb of code depending on how you optimize, expand, or secure it.
There are no new symbols, opcodes, metamethods. There's no problems with complex lvalue expressions.

### To-Do
This does not support tuple assignment yet.

```
This is Lua 5.4.4, released on 13 Jan 2022.

For installation instructions, license details, and
further information about Lua, see doc/readme.html.
```