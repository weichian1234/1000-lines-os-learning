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

## Exceptions in CPUs (RISC-V focus)
An exception is a CPU feature that lets the kernel handle special events like page faults, illegal instructions, and system calls. It works like a hardware-assisted try-catch—the CPU runs normally until intervention is needed. Unlike software try-catch, execution can often resume exactly where it left off after the kernel handles the exception.

Exceptions can also occur in kernel mode, often indicating serious bugs. Early implementation of an exception handler is recommended to gracefully crash (e.g., kernel panic), similar to adding global error handlers in JavaScript.

### Exception Handling Flow in RISC-V:

1. CPU checks medeleg to determine handler mode (usually S-mode via OpenSBI).

2. CPU saves state in Control and Status Registers (CSRs).

3. CPU jumps to exception handler via stvec.

4. The handler saves registers, processes the exception.

5. Handler restores state and uses sret to resume execution.

### Key CSRs involved:

- scause: Type of exception.

- stval: Extra details (e.g., memory address).

- sepc: Program counter at the exception point.

- sstatus: Mode (U/S) when exception occurred.

## Exception Handler (Entry Point for RISC-V Kernel Exception Handler)
### 🔧 Key Operations
1. Save the Stack Pointer:
    - sscratch is used to store the current stack pointer (sp) for restoration later.

2. Save All General-Purpose Registers:
    - All registers (ra, gp, tp, t0–t6, a0–a7, s0–s11) are pushed onto the stack.
    - Floating-point registers are not saved since they are unused by the kernel.

3. Call the Exception Handler:
    - a0 is set to point to the saved stack (register snapshot).
    - handle_trap(a0) is invoked to manage the trap using this saved context.

4. Restore Registers:
    - After handle_trap completes, all registers are restored from the stack in reverse order.
    - Finally, the stack pointer is restored, and the sret instruction returns from the trap.

### 📝 Notes
- __attribute__((naked)): Prevents compiler-generated prologue/epilogue code.
- __attribute__((aligned(4))): Ensures 4-byte alignment for use with stvec, whose lowest 2 bits are reserved for mode flags.
- Warning: Mishandling register saving/restoring—especially a0—can cause subtle, hard-to-debug issues. Careful attention is essential.