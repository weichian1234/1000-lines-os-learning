# Chapter 2: C Standard Library
In C programming, especially when dealing with systems programming or embedded development, a file named common.h is often used as a header file to define shared types, macros, and constants used throughout a project.

The "basic types" in a typical common.h file are usually typedefs or macro definitions that create standardized data types for portability and code clarity.

## common.h
- **paddr_t**: physical memory address 
- **vaddr_t**: virtual memory address. Equivalent to **uintptr_t**
- **align_up**: Rounds up value to the nearest multiple of "align". "align" must be a power of 2.
- **offsetof**: returns the offset of a member

memory alignment 
- align_up(0x1234, 0x1000) returns 0x2000.
- is_aligned(0x2000, 0x1000) returns true
- is_aligned(0x2f00, 0x1000) is false

