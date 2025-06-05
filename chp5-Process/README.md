# Chapter 5: Process
A process is an instance of an application

### Process Control Block (PCB)
Defines a process structure in an operating system. Key elements:
- Process ID (pid): Unique identifier for each process.
- State (state): Indicates whether a process is unused (PROC_UNUSED) or runnable (PROC_RUNNABLE).
- Stack Pointer (sp): Points to the current stack location.
- Kernel Stack (stack[8192]): Stores CPU registers, return addresses, and local variables for context switching.

### Context Switching
- Context switching allows the system to switch execution between processes by saving and restoring their state.

- Implementation Details
    - Function: switch_context(uint32_t *prev_sp, uint32_t *next_sp)
    - Saves callee-saved registers (s0 to s11, ra) onto the stack of the current process.
    - Updates the stack pointer (sp) to switch to the next process.
    - Restores registers from the new process’s stack to resume execution.

- Key Mechanisms
    - Callee-Saved Registers: Must be preserved across function calls.
    - Stack-Based Storage: Context is stored as temporary local variables in the stack rather than a structure.
    - naked Attribute: Prevents additional compiler-generated code, ensuring direct stack manipulation.
    - Calling Convention: Defines how registers are saved/restored.
- Process Initialization (create_process)
    - Finds an available process slot in procs[].

    - Initializes stack by preloading callee-saved registers with zero.

Sets process entry point (pc) in the return address (ra).

Marks process as PROC_RUNNABLE, setting its stack pointer.