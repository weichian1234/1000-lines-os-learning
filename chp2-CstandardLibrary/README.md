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


## memcpy
- uint8_t *d = (uint8_t *) dst;
    - 思考： why has (uint8_t *) in parentheses, but the variable name d does not.
    - (uint8_t *) → cast operator (must be in parentheses).
    - d → variable being declared (no parentheses needed).

## memset
- *p++ = c;
    - 思考：?
    - Pointer deferencing and pointer manipulation in a single statement. Equivalent to:
    - Dereference the pointer: *p = c;
    - Advance the pointer after the assignment: p = p + 1

## strcmp
- return *(unsigned char *)s1 - *(unsigned char *)s2;
    - 思考：why casting to unsigned char * when returning
    - conform to : The sign of a nonzero return value shall be determined by
              the sign of the difference between the values of the first
              pair of bytes (both interpreted as type unsigned char) that
              differ in the strings being compared. (https://www.man7.org/linux/man-pages/man3/strcmp.3.html#:~:text=both%20interpreted%20as%20type%20unsigned%20char)