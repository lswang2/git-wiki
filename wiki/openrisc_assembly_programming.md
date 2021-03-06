OpenRISC Assembly Programming
===============================

# Pure assembly programming

**test_asm.S**

        .section    .text.startup,"ax",@progbits
        .align  4
    .proc   main
        .global _start
        .global main
        .type   main, @function
    _start:
    main:
        l.sw        -4(r1),r1    # SI store
        l.addi      r1,r1,-4 # addsi3
    	l.nop
    while_loop:
    	l.j			while_loop

**compile**
    or1k-elf-as test_asm.S -o test_asm.o
    or1k-elf-ld test_asm.o -o test_asm.out

# Mixed C with assembly 

## function definition

**First make "add.c" for test**
~~~~
    int add(int a, int b)
    {
        return a+b;
    }
~~~~

**make assembly code**
~~~~
    or1k-elf-gcc -O2 -S asm.c
~~~~

**result "add.s"**

        .file   "add.c"
        .section .text
        .align  4
    .proc   add
        .global add
        .type   add, @function
    add:
    .LFB0: # ??
        .cfi_startproc
        # Stack pointer backup
        l.sw        -4(r1),r1    # SI store
        .cfi_offset 1, -4
        # Stack pointer move to new postion
        l.addi      r1,r1,-4 # addsi3
        .cfi_def_cfa_offset 4

        # body operation
        # arguments are r3,r4,...
        # return value is r11
        l.add       r11,r3,r4 # addsi3
    
        # rollback the stack pointer
        l.addi  r1,r1,4
        # return
        l.jr        r9  # return_internal   # delay slot filled
        # restore stack pointer
        l.lwz       r1,-4(r1)    # SI load
        .cfi_endproc
    .LFE0: # ??
        .size   add, .-add
        .ident  "GCC: (GNU) 5.3.0"

## inline assembly

**=r (target register), r (source register), n (number)**
~~~~
uint32_t check_flag(void)
{
    uint32_t flag;
    __asm__ __volatile__("l.mfspr\t\t%0,%1,0": "=r"(flag):"r"(0x0011));
    // l.mfspr   r4, 0x0011
    if(f!=((flag>>9)&0x01))
        return 0;
    else
        return 1;
}
~~~~
**no target register**
~~~~
__asm__ __volatile__("l.mtspr\t\t%0,%1,0": :"r"(0x001E), "r"(a));
// l.mtspr  0x0011, r4
~~~~
**constant numbers as arguments**
~~~~
__asm__ __volatile__("l.inst\t\t%0,%1 %2,%3 %4":"=r"(d_inst):"r"(a),"n"(1),"r"(b),"n"(1));
// l.inst r4, r1 0, r2 1
~~~~



