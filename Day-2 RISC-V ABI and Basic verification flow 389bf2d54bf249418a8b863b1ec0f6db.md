# Day-2 RISC-V ABI and Basic verification flow

Related Course(s): RISC-V Myth (https://www.notion.so/RISC-V-Myth-e52fd09cf2c142c9881dc05a0b7003a2?pvs=21)
Date Last Edited: September 23, 2023 11:56 AM
Summary: - Little Endian and Big Endian memory addressing system

## Level of Abstractions and various interfaces:

An application is created in an higher level programming language, which makes use of standard libraries offered by an Operating System, and functions accordingly. The Application Programming Interface (API) is the interface between the standard libraries and the application.

The OS then uses the ISA to convert the high-level code to instructions that can be understood by the architecture on which the OS is hosted.

The ISA of the processor architecture is implemented on an RTL design. The RTL will synthesize a logic circuit that would respond to an instruction and give outputs accordingly.

An application may directly access the registers of a processor, bypassing Operating System, with a System Call or Application Binary Interface (ABI).

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled.png)

<aside>
💡 Little Endian and Big Endian Memory Addressing System:

[Lecture 22. Big Endian and Little Endian](https://www.youtube.com/watch?v=T1C9Kj_78ek)

- **Little-Endian**: In little-endian systems, the least significant byte (LSB) of a multi-byte data item is stored at the lowest memory address, and the most significant byte (MSB) is stored at the highest memory address.
- **Big-Endian**: In big-endian systems, it's the opposite. The MSB is stored at the lowest memory address, and the LSB is stored at the highest memory address.

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%201.png)

</aside>

## ABI of a RISC-V architecture:

The interface has 32 registers of specific width. The width of the register is defined by `XLEN`, for RV64 XLEN=64 and for RV32 XLEN=32. Each register is used for different type of functionality or system call. 

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%202.png)

<aside>
💡 These registers are used for ABI purpose only. Since they are limited, 32 registers and each with different uses, they get filled very quickly and so we need to store them back in the memory right after the instruction executes. So we continuously load and store the registers.

</aside>

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%203.png)

In RISC V architecture, the instructions are 32-bits, even if we use the RV64 architecture.

1. Load Doubleword Instruction:
    
    `ld x8, 16(x23)` - This instruction will load the x8 register with the contents present at address given on x23+16. 16 is the offset given to the contents on x23. This offset is saved in “Immediate” bits.
    
    The structure of the instruction is shown. The opcode and funct3 bits will identify the keyword `ld` . Rd and Rs1 are destination and source registers respectively, which are of 5-bits.
    

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%204.png)

1. Add Instruction:
    
    `add x8, x24, x8` 
    

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%205.png)

1. Store Doubleword Instruction:

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%206.png)

## Algorithm using ASM language:

C language code:

```c
#include <stdio.h>
extern int load(int x, int y);
	int main() {
			int result = 0;
			int count = 9;
			result = load (0x0, count+1);
			printf("Sum of number from 1 to %d is %d\n", count, result);
}
```

ASM Code:

```nasm
.section .text
.global load
.type load, @function
load:   add a4, a0, zero //Initialize sum register a4 with 0x0
				add a2, a0, al   // store count of 10 in register a2. Register al is loaded with Oxa (decimal 10) from main
				add a3, a0, zero // initialize intermediate sum register a3 by 0
loop:		add	a4, a3, a4 	 // Incremental addition
				addi a3, а3, 1   // Increment intermediate register by 1
				blt	a3, a2, loop // If a3 is less than a2, branch to label named <loop>
				add	a0, a4, zero // Store final result to register a0 so that it can be read by main program
				ret
```

Parse the ASM code and the C language code together with command: `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1to9_custom.o 1to9_custom.c load.s` 

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%207.png)

Running a pre-wrriten code 1to9_custom.c on Picorv32, using iverilog. Every command to make this possible is written in the [rv32im.sh](http://rv32im.sh) file.

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%208.png)