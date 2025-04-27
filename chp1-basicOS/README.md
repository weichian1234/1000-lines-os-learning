# Chapter 1: Build the basic operating system

### Run the booting process.
-   Initialize the CPU and start executing the OS
- OS  initializes the hardware and starts the applications


## run.sh (CTRL + A then c -> debug )
- Set qemu file path = qemu-system-risv32


## linker script (kernel.ld)
- Memory layout of executable files
- Entry point of kernel = boot func()
- Base address = 0x80200000
- "." symbol represents the current address.
- The statement . += 128 * 1024 means "advance the current address by 128KB". 
- The ALIGN(4) directive ensures that the current address is adjusted to a 4-byte boundary.
- Each section is placed in the order of text, .rodata, .data, and .bss.
- The kernel stack comes after the .bss section, and its size is 128KB.
- __bss = . assigns the current address to the symbol __bss. In C language, you can refer to a defined symbol using extern char symbol_name.


## boot function attributes
-  __attribute__((naked)) : instruct compiler not to generate unecessary code before and after the function body, such as a return instruction. Ensure that hte inline assembly code is the exact function body
-    __attribute__((section(".text.boot"))): controls the placement of the function in the linker script. OepnSBI simply jumps, boot function needs to be placed at 0x80200000.

## extern char
- To obtain the address of the symbols
- extern char __bss[ ]:
    - without [ ], but __bss alone means "the value at the 0th byte of the .bss section" instead of "the start address of the .bss section".
    - with [ ], ensure that  __bss returns an address and prevent any careless mistakes.


## Debugging
- In QEMU-> info registers: get the info about the CPU regiters
- $ llvm-objdump -d kernel.elf -> to narrow the specific line of code
    - Address of instruction, Hexadecimal dump of the machine code, Disassembled instructions
- $ llvm-nm kernel.elf -> check the addresses of functions/variable
