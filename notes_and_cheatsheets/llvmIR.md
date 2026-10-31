source: https://simplifycpp.org/books/LLVM_IR_Quick_Reference.pdf

# LLVM IR - Low Level Virtual Machine Intermediate Representation

## Key Characteristics of LLVM IR

LLVM Intermediate Representation (IR) has five main characteristics:

1. **Static Single Assignment (SSA):**
   Each SSA value is assigned exactly once, simplifying
   data-flow analysis and optimizations such as constant
   propagation and dead code elimination.

2. **Strong Typing:**
   Every value and operation has a defined type, such as
   integers (`i32`), floating-point values (`float`),
   pointers (`ptr`), or structures.

3. **Three-Address Code:**
   Most instructions use two operands and produce one
   result, making optimization and machine-code generation
   easier.

4. **Extensible and Modular:**
   Supports metadata, extensions, and modular compiler
   passes, making it adaptable to different programming
   languages and applications.

5. **Human-Readable Format:**
   Available as readable text (`.ll`) and binary bitcode
   (`.bc`), useful for debugging, analysis, and compilation.



## Structure of LLVM IR

LLVM IR is organized into a hierarchy of seven components:

1. **Module:**
   Top-level compilation unit containing global variables,
   functions, and metadata.

2. **Function:**
   Callable unit containing parameters, a return type,
   attributes, and basic blocks.
   
   - **Function Attributes in LLVM IR**
   
        **Function attributes:** provide additional information about a function's behavior, helping the compiler perform optimizations.

            1. nounwind:
                Indicates that the function does not unwind the stack
                by throwing exceptions.

            2. readonly:
                Indicates that the function does not modify memory
                (but may read from it).

            3. inlinehint:
                Suggests that the compiler should inline the function,
                replacing the function call with its body.

        **Key Idea:** Function attributes describe a function's
                behavior and help LLVM optimize code safely and efficiently.


3. **Basic Block:**
   Sequence of instructions ending with a terminator
   (`ret`, `br`, `switch`). Basic blocks form the
   Control Flow Graph (CFG).

4. **Instruction:**
   Single operation, such as arithmetic (`add`),
   memory access (`load`, `store`), or control flow (`br`).

5. **Value:**
   Data used as an instruction operand, including
   constants, variables, and instruction results.

6. **Type:**
   Defines the kind of data, such as integers (`i32`),
   floating-point numbers (`float`), pointers (`ptr`),
   arrays, structures, and vectors.

7. **Metadata:**
   Additional information for debugging, profiling,
   or optimization that does not change program semantics.

### LLVM IR Hierarchy

```text
Module
 ├── Global Variables
 ├── Functions
 │    └── Basic Blocks
 │         └── Instructions
 │              └── Values (with Types)
 └── Metadata
```


## Advantages of LLVM IR

1. **Portability:**
   Platform-independent IR that supports multiple
   hardware architectures.

2. **Optimizability:**
   SSA and strong typing enable optimizations such as
   constant propagation, dead code elimination,
   and loop transformations.

3. **Interoperability:**
   Supports multiple languages (C, C++, Rust, Swift)
   and machine-code targets.

4. **Extensibility:**
   Supports custom compiler passes, metadata,
   and specialized extensions.

5. **Tooling Support:**
   Provides tools for optimization, debugging,
   and static analysis.

## Use Cases of LLVM IR

1. **Compilers:**
   Used by Clang, Rustc, and Swift compilers to
   generate and optimize machine code.

2. **JIT Compilation:**
   Generates and optimizes machine code at runtime
   (e.g., Julia).

3. **Static Analysis:**
   Detects bugs, security vulnerabilities,
   and performance issues.

4. **Domain-Specific Languages (DSLs):**
   Serves as a backend for custom programming languages.

5. **Research & Prototyping:**
   Used to experiment with compiler optimizations
   and program analysis.

## Example of LLVM IR

```llvm
define i32 @add(i32 %a, i32 %b) {
  %result = add i32 %a, %b
  ret i32 %result
}
```

**Explanation:**
- `define i32 @add(...)`: Defines a function returning
  a 32-bit integer.
- `%a`, `%b`: Two 32-bit integer parameters.
- `add i32 %a, %b`: Adds the parameters and assigns
  the result to `%result`.
- `ret i32 %result`: Returns the computed result.

## Conclusion

LLVM IR bridges high-level programming languages and
machine code. Its **SSA form, strong typing, portability,
and modular design** make it useful for compilation,
optimization, and static analysis.



## What is SSA (Static Single Assignment)?

**SSA** is a form of intermediate representation where each
variable is **assigned exactly once**. Every reassignment
creates a new variable with a unique name.

### Eg: Without SSA

```text
x = 1
x = x + 2
y = x * 3
```

Here, `x` is assigned twice.

### Eg: With SSA

```text
x1 = 1
x2 = x1 + 2
y1 = x2 * 3
```

- `x1 = 1`: First assignment.
- `x2 = x1 + 2`: New variable instead of modifying `x1`.
- `y1 = x2 * 3`: Uses the latest value of `x`.

**Key Idea:** In SSA, variables are never reassigned;
each new value gets a unique name. This simplifies
data-flow analysis and compiler optimizations.

