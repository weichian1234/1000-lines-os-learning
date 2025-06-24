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

### Mapping Pages (Two-Level Page Table Setup)
- Maps a virtual address (vaddr) to a physical address (paddr) using a two-level Sv32 page table.

- Ensures both vaddr and paddr are page-aligned.

- If the first-level page table entry (VPN[1]) is missing, it allocates a second-level table.

- Sets the second-level entry (VPN[0]) with physical page number and permission flags (PAGE_R, PAGE_W, etc.).

- Important detail: Physical page number (PPN) is stored, not the raw physical address.

### Mapping Kernel Memory
- Kernel uses identity mapping: virtual and physical addresses are identical (vaddr == paddr) to simplify early access.

- This mapping spans from __kernel_base to __free_ram_end, ensuring access to both code and dynamically allocated memory.

- The kernel's base address is defined in kernel.ld and must follow . = 0x80200000 to be correct.

- The process struct is extended to include a pointer to its page table.

### Switching page table (in yield())

- Before switching contexts:
    - The page table pointer is written to the satp register.
    - sfence.vma is called before and after to:
        - Ensure memory ordering.
        - Invalidate old TLB entries.
- sscratch is also updated with the new kernel stack base.