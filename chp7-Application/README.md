# Chapter 7: Application


### Memory Layout (New Linker Script user.ld)
- Isolated Address Space: The application is loaded at virtual address 0x1000000, avoiding overlap with kernel memory.
- Section Layout:
    - .text → contains executable code.
    - .rodata → read-only constants.
    - .data → initialized global/static variables.
    - .bss → zero-initialized data and a 64KB stack space (__stack_top marker).
- Memory Size Guard: An ASSERT ensures the total memory used stays below 0x1800000, preventing oversized executables.

### USerland Library Initialization
- start Function:
    - Entry point for the application.
    - Sets up the stack pointer (sp) using __stack_top.
    - Invokes main() and, afterward, calls exit().
    - Marked as naked and placed in .text.start to ensure low-level control and predictable layout.

- exit Function:
    - Currently implemented as an infinite loop to simulate program termination.

- putchar Stub:
    - Placeholder for output functionality used by printf.
    - To be implemented later.

- Zero-Initialization:
    - Unlike the kernel, the application doesn’t clear .bss explicitly—this is handled safely during memory allocation (alloc_pages() ensures zeroed memory).

- user.h Header:
    - Declares interfaces for exit() and putchar(), and includes shared definitions via common.h.

