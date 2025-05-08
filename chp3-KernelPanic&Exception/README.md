# Chapter 3: Kernel Panic and Exception
| Aspect             | Exception                                       | Kernel Panic                        |                                                                                                                           |
| ------------------ | ----------------------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Severity**       | Often recoverable                               | Critical, non-recoverable           |                                                                                                                           |
| **Scope**          | Affects specific applications or processes      | Affects the entire operating system |                                                                                                                           |
| **Handling**       | Managed by exception handlers; system continues | System halts; requires reboot       |                                                                                                                           |
| **Cause Examples** | Invalid operations, access violations           | Hardware failures, kernel bugs      | 

## Kernel Panic (Breakdown of the code)
- #define PANIC(fmt, ...) 
    - Macro Definition with Variadic Arguments: defines a macro named PANIC that accepts a format string fmt and a variable number of additional arguments (...).
- __VA_ARGS__
    - This is a useful compiler extension for defining macros that accept a variable number of arguments. ## removes the preceding , when the variable arguments are empty. This allows compilation to succeed even when there's only one argument, like PANIC("booted!").
    
- do { ... } while (0) Construct:
    - Ensures that the macro behaves like a single statement, allowing it to be used safely in control structures without expected behavior.

- Infinite Loop (while (1) {}):
    - After printing the error message, the macro enters an infinite loop, effectively halting program execution.
