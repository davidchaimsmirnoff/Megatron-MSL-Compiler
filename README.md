
# 🧊⚙️ Arctic Metal Compiler 
 
**A GPU compiler REPL that compiles a tiny expression language into Metal Shading Language (MSL) and runs it on the GPU.**
 
Write `out = a + b`. The Arctic Compiler lexes it, parses it into an AST, generates a Metal kernel, compiles it on the fly, and dispatches it to your GPU — all in one command.
 
---
 
## Features
 
- **Expression → GPU**: Arithmetic expressions over `a`, `b` buffer inputs are compiled to MSL kernels
- **REPL interface**: Interactive shell for defining and running kernels
- **Live Metal compilation**: Kernels are compiled at runtime via `MTLLibrary`
- **Named kernel registry**: Store multiple kernels and run them by name
- **Full compiler pipeline**: Lexer → Parser → AST → Codegen → Metal
---
 
## Architecture
 
```
Input expression
      │
      ▼
  ┌─────────┐
  │  MLexer │   Tokenizes identifiers and operators
  └────┬────┘
       │ tokens
       ▼
  ┌─────────┐
  │ MParser │   Builds an AST with operator precedence (* / before + -)
  └────┬────┘
       │ MAssign AST node
       ▼
  ┌──────────┐
  │ MCodegen │   Walks the AST and emits Metal Shading Language source
  └────┬─────┘
       │ MSL string
       ▼
  ┌──────────┐
  │ MRuntime │   Compiles and dispatches the kernel on the GPU via Metal
  └──────────┘
```
 
### Source Files
 
| File | Role |
|---|---|
| `Lexer.h / .m` | Tokenizes expression strings into `ID` and `OP` tokens |
| `Parser.h / .m` | Recursive-descent parser; produces `MAssign` AST nodes |
| `AST.h / .m` | AST node types: `MNode`, `MVar`, `MBinOp`, `MAssign` |
| `Codegen.h / .m` | Walks AST nodes and emits MSL kernel source |
| `MRuntime.h / .m` | Metal device setup, kernel compilation, and GPU dispatch |
| `MKernel.h / .m` | Data model for a named kernel (name + source) |
| `MKernelRegistry.h / .m` | In-memory dictionary mapping kernel names to `MKernel` objects |
| `main.m` | REPL loop; dispatches `kernel`, `run`, `help`, and `exit` commands |
 
---
 
## Requirements
 
- **macOS** with a Metal-capable GPU (Apple Silicon or AMD/Intel discrete)
- **Objective-C** (Foundation + Metal frameworks)
---
 
## Building
 
```bash
clang -fobjc-arc \
  main.m Lexer.m Parser.m AST.m Codegen.m \
  MRuntime.m MKernel.m MKernelRegistry.m \
  -framework Foundation -framework Metal \
  -o megatron
 
./megatron
```
 
---
 
## Usage
 
```
🔥 Arctic GPU Compiler REPL v0.1
Type 'help' for usage.
 
megatron >
```
 
### Commands
 
#### `kernel <name> <expression>`
 
Compiles an arithmetic expression into a Metal kernel and stores it by name.
 
```
megatron > kernel add out = a + b
megatron > kernel mul out = a * b
megatron > kernel fma out = a * b + a
```
 
The expression must be an assignment to `out`. The variables `a` and `b` are bound to GPU input buffers.
 
#### `run <name> <a> <b>`
 
Executes a previously compiled kernel with two float inputs.
 
```
megatron > run add 3.0 4.0
✅ Result: 7.000000
 
megatron > run mul 3.0 4.0
✅ Result: 12.000000
```
 
#### `help`
 
Prints usage information.
 
#### `exit`
 
Quits the REPL.
 
---
 
## Example Session
 
```
megatron > kernel add out = a + b
✅ Kernel 'add' compiled and stored.
--- Generated MSL ---
#include <metal_stdlib>
using namespace metal;
 
kernel void compute(device float* out [[buffer(0)]],
                    device float* a [[buffer(1)]],
                    device float* b [[buffer(2)]],
                    uint id [[thread_position_in_grid]]) {
    out[id] = (a[id] + b[id]);
}
---------------------
 
megatron > run add 10.0 5.0
✅ Result: 15.000000
 
megatron > kernel sub out = a - b
✅ Kernel 'sub' compiled and stored.
 
megatron > run sub 10.0 3.0
✅ Result: 7.000000
 
megatron > exit
👋 bye.
```
 
---
 
## Expression Language
 
The language supports:
 
- **Variables**: `a`, `b` (bound to Metal input buffers), `out` (output buffer)
- **Binary operators**: `+`, `-`, `*`, `/`
- **Operator precedence**: `*` and `/` bind tighter than `+` and `-`
- **Grouping**: Implicit via precedence; explicit parentheses not yet supported
Expressions are assigned to `out`:
 
```
out = a + b * a
```
 
---
 
## Limitations & Roadmap
 
- [ ] Only `a` and `b` input variables are supported (no literals/constants)
- [ ] Single-element dispatch only (no vector/batch processing beyond index 0)
- [ ] No parenthesized grouping in expressions
- [ ] No error recovery in the parser
- [ ] Kernel persistence (registry is in-memory only, lost on exit)
- [ ] No support for user-defined functions or multi-statement kernels
---
 
## License
 
MIT
