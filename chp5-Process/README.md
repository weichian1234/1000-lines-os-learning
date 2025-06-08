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

    - Sets process entry point (pc) in the return address (ra).

    - Marks process as PROC_RUNNABLE, setting its stack pointer.

### Scheduler
In the previous experiment, we directly called the switch_context function to specify the "next process to execute". However, this method becomes complicated when determining which process to switch to next as the number of processes increases. To solve this issue, this implementation introduces a scheduler (yield()), which selects the next process to run based on predefined criteria.

- Implementation Details
    - Global Variables:
        - current_proc: Tracks the currently running process.
        - idle_proc: Represents an idle process, running when no other process is available.

    - yield() Function:
        - Iterates through processes, searching for one in the PROC_RUNNABLE state.
        - If no runnable process is found, it continues execution.
        - Otherwise, it switches context to the selected process using switch_context().

- Idle Process Initialization (kernel_main())
    - The idle process (PID = 0) is created and assigned to current_proc at boot.
    - This ensures proper execution context management when switching processes.
- Process Execution (proc_a_entry & proc_b_entry)
    - Instead of directly calling switch_context(), processes voluntarily yield CPU time using yield().
    - Result: Alternating execution between Process A and Process B, ensuring cooperative multitasking.

### Exception Handler Updates
Since each process now has its own independent kernel stack, the exception handler must be modified to properly manage execution state.

- Key Changes
    - Storing Kernel Stack in sscratch
        - During context switching, the bottom of the kernel stack for the next process is stored in the sscratch register.
        - This ensures the exception handler can correctly retrieve the process's stack.
    - Modifications in kernel_entry()
        - Swaps the stack pointer (sp) with sscratch, switching to the kernel stack.
        - Saves 31 registers onto the stack, ensuring state preservation.
        - Retrieves the original user stack pointer (sp) from sscratch before handling the exception.
        - Restores the previous state after handling, resuming execution seamlessly.
    - Nested Interrupt Handling
        - The implementation does not support nested interrupts.
        - The CPU automatically disables interrupts upon entering stvec (kernel_entry).
        - A possible improvement would be separate handlers for user mode and kernel mode.
    - Context Switching Impact
        - By updating sscratch during context switching, each process resumes exactly where it was interrupted, maintaining continuity without explicit tracking.

### Why Reset the Stack Pointer?
During an exception, trusting the stack pointer (sp) can be risky, especially when an exception occurs in user mode. To ensure safe execution, the system must switch to a trusted kernel stack using sscratch.

- Three Exception Cases
    1. Kernel Mode Exception → Safe, does not require stack resetting.
    2. Nested Kernel Exception → Could overwrite the stack, but triggers a kernel panic, so it's manageable.
    3. User Mode Exception → Problematic, as sp could point to an invalid or unmapped memory address.
- Vulnerability Risk
    - If an exception occurs with an invalid sp (e.g., 0xdeadbeef), the system attempts to save registers on an inaccessible stack.
    - This leads to an infinite loop where the trap handler continuously triggers itself, hanging the kernel.
- Solution Approaches
    - Using sscratch: Stores a valid kernel stack during context switching.
    - Separtate Exception Handlers:
        - kernelvec for kernel mode exceptions (inherits existing stack).
        - uservec for user mode exceptions (switches to a separate kernel stack).
        - Used in RISC-V xv6 to prevent execution failures.
- Security Lesson
    - Past vulnerabilities (e.g., in Google's Fuchsia OS) show why trusting user inputs—including registers and memory pointers—is dangerous. Proper exception handling ensures robustness in kernel development.
