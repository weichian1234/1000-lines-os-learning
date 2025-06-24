# Chapter 6: Page Table

### Memory Management and Virtual Addressing
When a program accesses memory, it uses virtual addresses, which the CPU translates into physical addresses using a page table. This mapping enables:

- Memory isolation: Each process has its own virtual address space.
- Security: Kernel and application memory are kept separate.
- Flexibility: The same virtual address can point to different physical locations depending on the active page table.

The focus of the chapter is to implement hardware-based memory isolation, laying the groundwork for secure and compartmentalized execution environments.

### Structure of virtual address
The Sv32 paging scheme in RISC-V uses a two-level page table for translating virtual addresses to physical addresses. Here's how a 32-bit virtual address is broken down:

- VPN[1] (10 bits): Index into the first-level page table.
- VPN[0] (10 bits): Index into the second-level page table.
- Offset (12 bits): Position within the 4KB page.

#### Key Insights
- Page locality: Changes in VPN[0] affect only the second-level index—nearby virtual addresses stay within the same top-level entry.

- Page alignment: Changes in the 12-bit offset don’t change the virtual page mapping—data within the same 4KB page shares one page table entry.

- This design supports efficient memory mapping and leverages the TLB for faster address translation.

### Constructing the page table
- Macro Overview:
    - SATP_SV32:
        - Bit 31 of the satp register.
        - When set, it enables Sv32 mode for virtual memory translation.

    - PAGE_V (Valid):
        - Enables the page table entry (PTE). Must be set for any PTE to be considered.

    - PAGE_R (Readable), PAGE_W (Writable), PAGE_X (Executable):
        - Standard access permission bits.
        - These determine what operations are allowed on the mapped page.
    - PAGE_U (User Accessible):
        - Marks the page as accessible in user mode (as opposed to kernel-only access).