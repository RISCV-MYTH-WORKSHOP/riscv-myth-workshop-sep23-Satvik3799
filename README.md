# Table of Contents
- [Day-1 RISC-V ISA](#Day-1-RISC-V-ISA)
- [Day-2 RISC-V ABI and Basic verification flow](#Day-2-RISC-V-ABI-and-Basic-verification-flow)
- [Day-3 Digital Logic with TL-Verilog and Makerchip](#Day-3-Digital-Logic-with-TL-Verilog-and-Makerchip)
- [Day-4 RISC V Microarchitecture](#Day-4-RISC-V-Microarchitecture)
- [Day-5 Complete Pipelined RISC-V CPU micro-architecture](#Day-5-Complete-Pipelined-RISC-V-CPU-micro-architecture)

# Day-1 RISC-V ISA

Summary: - RISC-V GCC compiler
- SPIKE Debugger
- RISC-V ISA

## Using the RISC-V GCC compiler to compile a code written in C language.

Commands used:

1. write a C program in leafpad, to sum 1 to n numbers.
    
    ```c
    //File name is sum1ton.c
    #include <stdio.h>
    
    int main() {
    
      int i, sum = 0, n = 15;
      for (i=1; i<=n; ++i ){
      sum += 1;
      }
      printf("Sum of numbers from 1 to %d is %d",n,sum);
      return 0;
    }
    ```
    
2. Compiled using GCC.
3. Compiling using RISC-V GCC compiler with the command:
    
    It is used to compile C source code into an executable binary for the RISC-V architecture.
    
    General command - `riscv64-unknown-elf-gcc <compiler option -O1 ; Ofast> <ABI specifier -lp64; -lp32; -ilp32> <architecture specifier-RV64; RV32> -o <object-filename> <C-filename>`
    
    - The **`riscv64-unknown-elf`** prefix is typically used for cross-compilation, where you compile code on one architecture (e.g., x86) to run on another (e.g., RISC-V).
    - `march=ISA` selects the architecture to target. This controls which instructions and registers are available for the compiler to use. here we used, `march=rv64i` which means the general-purpose integer registers are 64-bits wide.
    - `mabi=ABI` selects the ABI to target. This controls the calling convention (which arguments are passed in which registers) and the layout of data in memory. Here we used, `mabi=lp64` which means `long` and pointers are 64-bits long, while `int` is a 32-bit type. The other types remain the same as ilp32.
    
    <aside>
    💡 The "long" data type is used to represent integers with a larger range of values compared to the standard "int" data type.
    
    </aside>
    
    - `mtune=CODENAME` selects the microarchitecture to target. This informs GCC about the performance of each instruction, allowing it to perform target-specific optimizations.
    - `-o sum1ton.o`This specifies the output file name for the compiled object file.
    
    Command used here -  `riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c`
    
    This will generate a set of instructions or assembly language code needed for the code to be run on a RISC-V machine.
    
    - To view assembly code use the below command, where `-d` stands for **********************disassemble********************** `riscv64-unknown-elf-objdump -d <object filename>`
    
    We are looking for address of the `<main>` block of the code we have written. It is stored at **x'10184,** and it uses (x’101b0-x’10184)/4 = x’B or $(11)_d$ addresses to save the needed instructions. Why it leaps 4 bits?
    
    ![Untitled](Day-1%20RISC-V%20ISA%201e5440d16f0b4c3289a57fff4d8b8e16/Untitled.png)
    
    When used `-Ofast`By using the command: `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c` 
    
    - Hte **`-Ofast`** option in GCC is a high-level optimization level that includes aggressive optimization settings for code performance. When you use the **`-Ofast`** option with the RISC-V GCC compiler, it instructs the compiler to apply a wide range of optimizations that can potentially lead to faster code execution but may also make certain trade-offs that could impact code behavior.
    
    ![Untitled](Day-1%20RISC-V%20ISA%201e5440d16f0b4c3289a57fff4d8b8e16/Untitled%201.png)
    
    More on options that can be used to run the compiler can be found here:
    
    [https://www.sifive.com/blog/all-aboard-part-1-compiler-args](https://www.sifive.com/blog/all-aboard-part-1-compiler-args)
    

## Debugging using SPIKE:

SPIKE is a RISC-V ISA (Instruction Set Architecture) simulator developed by the RISC-V project. It serves as a reference simulator for RISC-V, designed to aid in the development and testing of RISC-V software and hardware.

To use SPIKE simualtor to run risc-v obj file use the command: `spike pk <object filename>`

To use SPIKE as debugger `spike -d pk <object Filename>` with some debug commands as

 `until pc 0 <pc of your choice>` - To go to a particular address, till the instructions have been executed.

`reg 0 a0` will show the contents of the register at the moment. When pressing “Enter”, the next instruction in the PC would execute and the corresponding registers will get updated.

The PC was at x’100b0, and after executing the instruction present at x’100b0, which is `lui a0, 0x21` , it updated the register accordingly.

<aside>
💡 LUI - Load Upper Immediate. It will load with the hex value given to the destination register. But only bits 32:12 are updated and puts zero to the rest of the LSBs up to 12.

</aside>

![Untitled](Day-1%20RISC-V%20ISA%201e5440d16f0b4c3289a57fff4d8b8e16/Untitled%202.png)

## 64-bit Number Representation:

C language code;

```c
#include <stdio.h>
#include <math.h>

int main () {

// Any value above or below the limit would result in the highest or lowest number it can represent, so 2^127 is chosen randomly.

//For int

int max_i = (int) (pow(2,127)-1);
printf("highest number represented by int is %d\n", max_i);

int min_i = (int) (pow(2,127)*-1);
printf("Lowest number represented by int is %d \n", min_i);

// For Unsigned integer

unsigned int max_ui = (unsigned int) (pow(2,127)-1);
printf("highest number represented by unsigned is %u\n", max_ui);

unsigned int min_ui = (unsigned int) (pow(2,127)*-1);
printf("Lowest number represented by unsigned int is %u \n", min_ui);

//For long long int

long long int max_lld = ( long long int) (pow(2,127)-1);
printf("highest number represented by int is %lld\n", max_lld);

long long int min_lld = (long long int) (pow(2,127)*-1);
printf("Lowest number represented by int is %lld \n", min_lld);

// For Unsigned integer

unsigned int max_llu = (unsigned long long int) (pow(2,127)-1);
printf("highest number represented by unsigned long long int is %llu\n", max_llu);

unsigned int min_llu = (unsigned long long int) (pow(2,127)*-1);
printf("Lowest number represented by unsigned long long int is %u \n", min_llu);

}
```

![Untitled](Day-1%20RISC-V%20ISA%201e5440d16f0b4c3289a57fff4d8b8e16/Untitled%203.png)

![Untitled](Day-1%20RISC-V%20ISA%201e5440d16f0b4c3289a57fff4d8b8e16/Untitled%204.png)


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
loop:    add   a4, a3, a4   // Incremental addition
            addi a3, а3, 1   // Increment intermediate register by 1
            blt   a3, a2, loop // If a3 is less than a2, branch to label named <loop>
            add   a0, a4, zero // Store final result to register a0 so that it can be read by main program
            ret
```

Parse the ASM code and the C language code together with command: `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1to9_custom.o 1to9_custom.c load.s` 

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%207.png)

Running a pre-wrriten code 1to9_custom.c on Picorv32, using iverilog. Every command to make this possible is written in the [rv32im.sh](http://rv32im.sh) file.

![Untitled](Day-2%20RISC-V%20ABI%20and%20Basic%20verification%20flow%20389bf2d54bf249418a8b863b1ec0f6db/Untitled%208.png)


# Day-3 Digital Logic with TL-Verilog and Makerchip

## Ex. 1 - Validity Tutorial

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled.png)

### Combinational Logic

1. Inverter:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%201.png)

1. 2-input NAND :
    
    ![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%202.png)
    
2. 2-input NOR:
    
    ![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%203.png)
    
3. 2-input XOR:
    
    ![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%204.png)
    

### Ex. 3 -  Vectors:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%205.png)

### Ex. 4 - Mux:

Single bit:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%206.png)

Vector:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%207.png)

### Ex. 5 - Math operators with 4to1 MUX:

Logic used:

```nasm
$val1[31:0] = $rand1[3:0];
   $val2[31:0] = $rand2[3:0];
   
   $sum[31:0] = $val1[31:0] + $val2[31:0];
   $diff[31:0] = $val1[31:0] - $val2[31:0];
   $prod[31:0] = $val1[31:0] * $val2[31:0];
   $quo[31:0] = $val1[31:0] / $val2[31:0];
   
   $out[31:0] = $sel[0] ? $sum[31:0] : ($sel[1] ? $diff[31:0] : ($sel[2] ? $prod[31:0] : $quo[31:0]));
```

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%208.png)

## Sequential Logic

### Ex. 6 - Free running counter

Logic used:

`$counter[15:0] = $reset ? 0 : (1 + >>1$counter[15:0] );`

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%209.png)

### Ex. 7 - Fibonacchi Series;

Logic used: `$Fibo[31:0] = $reset ? 1 : (>>1$Fibo + >>2$Fibo );`

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2010.png)

### PipeLine Solution:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2011.png)

###2 cycle calc:

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/stevehoover/RISC-V_MYTH_Workshop/ecba3769fff373ef6b8f66b3347e8940c859792d/tlv_lib/calculator_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)

\TLV
   |calc
      @1
         $reset = *reset;
         
         $val1[31:0] = >>2$out[31:0];
         $val2[31:0] = $rand2[3:0];
   
         $sum[31:0] = $val1[31:0] + $val2[31:0];
         $diff[31:0] = $val1[31:0] - $val2[31:0];
         $prod[31:0] = $val1[31:0] * $val2[31:0];
         $quot[31:0] = $val1[31:0] / $val2[31:0];
         
         $cnt[0] = $reset ? 0 : (1 + >>1$cnt[0]);
   
      @2
         $valid[0] = $cnt[0];
         $out[31:0] = ($reset + ! $valid[0]) ? 0 : 
                      (($op[1:0] == 2'b00) ? $sum[31:0] : 
                        (($op[1:0] == 2'b01) ? $diff[31:0] : 
                           (($op[1:0] == 2'b10) ? $prod[31:0] : $quot[31:0])));
         

      // Macro instantiations for calculator visualization(disabled by default).
      // Uncomment to enable visualisation, and also,
      // NOTE: If visualization is enabled, $op must be defined to the proper width using the expression below.
      //       (Any signals other than $rand1, $rand2 that are not explicitly assigned will result in strange errors.)
      //       You can, however, safely use these specific random signals as described in the videos:
      //  o $rand1[3:0]
      //  o $rand2[3:0]
      //  o $op[x:0]
      
   //m4+cal_viz(@3) // Arg: Pipeline stage represented by viz, should be atleast equal to last stage of CALCULATOR logic.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   

\SV
   endmodule
```

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2012.png)

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2013.png)

link - 

[Makerchip](https://makerchip.com/sandbox/0M8f5hkL7/0wjh6B#)

### 2-cycle calc with valid:

```nasm
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/stevehoover/RISC-V_MYTH_Workshop/ecba3769fff373ef6b8f66b3347e8940c859792d/tlv_lib/calculator_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)

\TLV
   |calc
      @0
        $reset = *reset; 
      @1
         $valid_or_reset = $valid || $reset;
         
         $valid[0] = $reset ? 0 : (1 + >>1$valid[0]);
         $val1[31:0] = >>2$out[31:0];
         $val2[31:0] = $rand2[3:0];
         
      ?$valid_or_reset
         @1
            $sum[31:0] = $val1[31:0] + $val2[31:0];
            $diff[31:0] = $val1[31:0] - $val2[31:0];
            $prod[31:0] = $val1[31:0] * $val2[31:0];
            $quot[31:0] = $val1[31:0] / $val2[31:0];

         @2
            $out[31:0] = $reset ? 0 : 
              (($op[1:0] == 2'b00) ? $sum[31:0] : 
                (($op[1:0] == 2'b01) ? $diff[31:0] : 
                    (($op[1:0] == 2'b10) ? $prod[31:0] : $quot[31:0])));
         

      // Macro instantiations for calculator visualization(disabled by default).
      // Uncomment to enable visualisation, and also,
      // NOTE: If visualization is enabled, $op must be defined to the proper width using the expression below.
      //       (Any signals other than $rand1, $rand2 that are not explicitly assigned will result in strange errors.)
      //       You can, however, safely use these specific random signals as described in the videos:
      //  o $rand1[3:0]
      //  o $rand2[3:0]
      //  o $op[x:0]
      
   m4+cal_viz(@3) // Arg: Pipeline stage represented by viz, should be atleast equal to last stage of CALCULATOR logic.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   

\SV
   endmodule
```

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2014.png)

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2015.png)

Link-

[Makerchip](https://makerchip.com/sandbox/0M8f5hkL7/0xGh87#)

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2016.png)


# Day-4 RISC V Microarchitecture

Related Course(s): RISC-V Myth (https://www.notion.so/RISC-V-Myth-e52fd09cf2c142c9881dc05a0b7003a2?pvs=21)
Date Last Edited: September 24, 2023 10:20 PM

## PC Counter

Notice, the PC becomes 0, one clock cycle after reset = 1. We want to execute the instruction 0, if we immediately do the PC=0, when reset=1, it will execute instruction 1, because we add one after the PC gets its value.

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled.png)

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%201.png)

## Fetch

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = (>>1$reset) ? '0 : >>1$pc + 32'd4;
         
         $imem_rd_en = !>>1$reset ? 1 : 0;
         $imem_rd_addr[31:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
            
      @1         
         $instr[31:0] = $imem_rd_data[31:0];

      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      //m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

      //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%202.png)

## Decode

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%203.png)

### Decoding instructions IRSJBU

```verilog

\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = (>>1$reset) ? '0 : >>1$pc + 32'd4;
         
         $imem_rd_en = !>>1$reset ? 1 : 0;
         $imem_rd_addr[31:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
            
      @1         
         $instr[31:0] = $imem_rd_data[31:0];
         
			//Decoding instructions IRSBJU
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] == 5'b11001;
         
         $is_r_instr = $instr[6:2] ==? 5'b01x1x ||
                       $instr[6:2] ==? 5'bxx100;
                       
         $is_b_instr = $instr[6:2] == 5'b11000;
         $is_u_instr = $instr[6:2] == 5'b0x101;
         $is_s_instr = $instr[6:2] == 5'b0100x;
         $is_j_instr = $instr[6:2] == 5'b11011;

				
      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      //m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

      //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%204.png)

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%205.png)

Notice, the PC = (x0000_0018), in the next clock cycle instruction B gets enabled.

### Decoding immediate

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = (>>1$reset) ? '0 : >>1$pc + 32'd4;
         
         $imem_rd_en = !>>1$reset ? 1 : 0;
         $imem_rd_addr[31:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
            
      @1         
         $instr[31:0] = $imem_rd_data[31:0];
         
         //Decoding Instructions IRSBJU
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] == 5'b11001;
         
         $is_r_instr = $instr[6:2] ==? 5'b01x1x ||
                       $instr[6:2] ==? 5'bxx100;
                       
         $is_b_instr = $instr[6:2] == 5'b11000;
         $is_u_instr = $instr[6:2] == 5'b0x101;
         $is_s_instr = $instr[6:2] == 5'b0100x;
         $is_j_instr = $instr[6:2] == 5'b11011;
         
         //Decoding Immediate
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}} , $instr[30:20] } :
                   $is_s_instr ? { {21{$instr[31]}} , $instr[30:25] , $instr[11:8] , $instr[7] } :
                   $is_b_instr ? { {20{$instr[31]}} , $instr[7] , $instr[30:25] , $instr[11:8] , 1'b0} :
                   $is_u_instr ? { $instr[31] , $instr[30:20] , $instr[19:12] , 12'b0} : 
                   $is_j_instr ? { {12{$instr[31]}} , $instr[19:12] , $instr[20] , $instr[30:21] , 1'b0} : 32'b0;

      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      //m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

      //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%206.png)

### Decode other Fields of Instructions For RV-ISBUJ (Also used a When condition, to only define when the instruction has such a field)

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = (>>1$reset) ? '0 : >>1$pc + 32'd4;
         
         $imem_rd_en = !>>1$reset ? 1 : 0;
         $imem_rd_addr[31:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
            
      @1         
         $instr[31:0] = $imem_rd_data[31:0];
         
         //Decoding Instructions IRSBJU
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] == 5'b11001;
         
         $is_r_instr = $instr[6:2] ==? 5'b01x1x ||
                       $instr[6:2] ==? 5'bxx100;
                       
         $is_b_instr = $instr[6:2] == 5'b11000;
         $is_u_instr = $instr[6:2] == 5'b0x101;
         $is_s_instr = $instr[6:2] == 5'b0100x;
         $is_j_instr = $instr[6:2] == 5'b11011;
         
         //Decoding Immediate
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}} , $instr[30:20] } :
                   $is_s_instr ? { {21{$instr[31]}} , $instr[30:25] , $instr[11:8] , $instr[7] } :
                   $is_b_instr ? { {20{$instr[31]}} , $instr[7] , $instr[30:25] , $instr[11:8] , 1'b0} :
                   $is_u_instr ? { $instr[31] , $instr[30:20] , $instr[19:12] , 12'b0} : 
                   $is_j_instr ? { {12{$instr[31]}} , $instr[19:12] , $instr[20] , $instr[30:21] , 1'b0} : 32'b0;
         
         //Other fields of Instruction with a when condition
         $rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         $rs1_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         $rd_valid = $is_r_instr || $is_i_instr || $is_u_instr || $is_j_instr;
         $funct7_valid = $is_r_instr;
         $funct3_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
         $opcode[6:0] = $instr[6:0];
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      //m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

      //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%207.png)

## Register Read:

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = (>>1$reset) ? '0 : >>1$pc + 32'd4;
         
         $imem_rd_en = !>>1$reset ? 1 : 0;
         $imem_rd_addr[31:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
            
      @1         
         $instr[31:0] = $imem_rd_data[31:0];
         
         //Decoding Instructions IRSBJU
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] == 5'b11001;
         
         $is_r_instr = $instr[6:2] ==? 5'b01x1x ||
                       $instr[6:2] ==? 5'bxx100;
                       
         $is_b_instr = $instr[6:2] == 5'b11000;
         $is_u_instr = $instr[6:2] == 5'b0x101;
         $is_s_instr = $instr[6:2] == 5'b0100x;
         $is_j_instr = $instr[6:2] == 5'b11011;
         
         //Decoding Immediate
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}} , $instr[30:20] } :
                   $is_s_instr ? { {21{$instr[31]}} , $instr[30:25] , $instr[11:8] , $instr[7] } :
                   $is_b_instr ? { {20{$instr[31]}} , $instr[7] , $instr[30:25] , $instr[11:8] , 1'b0} :
                   $is_u_instr ? { $instr[31] , $instr[30:20] , $instr[19:12] , 12'b0} : 
                   $is_j_instr ? { {12{$instr[31]}} , $instr[19:12] , $instr[20] , $instr[30:21] , 1'b0} : 32'b0;
         
         //Other fields of Instruction with a when condition
         $rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         $rs1_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         $rd_valid = $is_r_instr || $is_i_instr || $is_u_instr || $is_j_instr;
         $funct7_valid = $is_r_instr;
         $funct3_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
         $opcode[6:0] = $instr[6:0];
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
            
         //decode individual instructions
         $dec_bits[10:0] = {$funct7[5],$funct3,$opcode};
         $is_beq = $dec_bits ==? 11'bx_000_1100011;
         $is_bne = $dec_bits ==? 11'bx_001_1100011;
         $is_blt = $dec_bits ==? 11'bx_100_1100011;
         $is_bge = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu = $dec_bits ==? 11'bx_111_1100011;
         $is_addi = $dec_bits ==? 11'bx_000_0010011;
         $is_add = $dec_bits ==? 11'b0_000_0110011;
         
         
         //Register File Read
         $rf_rd_en1 = $rs1_valid;
         $rf_rd_en2 = $rs2_valid;
         $rf_rd_index1[4:0] = $rs1;
         $rf_rd_index2[4:0] = $rs2;
         
         $src1_value[31:0] = $rf_rd_data1;
         $src2_value[31:0] = $rf_rd_data2;

         
    
         `BOGUS_USE($is_beq $is_bne $is_blt $is_bge $is_bltu $is_bgeu $is_addi $is_add)
      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

      //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%208.png)

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%209.png)

See, on reset=1, after 1 clock cycle, Reg 5 is set to 32’d5, which can be seen here.

## Added 

1. ALU Operations for ADD and ADDI instructions
2. Branch Instructions
3. Register File Read 
4. Register File write
5. Edited PC Counter to include branch instructions

To check the simulation, 

`*passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9);`

and verify with the value in XREG[10] = x2D or d45

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = >>1$reset ? 32'd0 : 
            >>1$taken_br ? >>1$br_tgt_pc :
            (>>1$pc + 32'd4);

         $imem_rd_en = !>>1$reset ? 1 : 0;
         $imem_rd_addr[31:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
            
      @1         
         $instr[31:0] = $imem_rd_data[31:0];
         
         //Decoding Instructions IRSBJU
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] == 5'b11001;
         
         $is_r_instr = $instr[6:2] ==? 5'b01x1x ||
                       $instr[6:2] ==? 5'bxx100;
                       
         $is_b_instr = $instr[6:2] == 5'b11000;
         $is_u_instr = $instr[6:2] == 5'b0x101;
         $is_s_instr = $instr[6:2] == 5'b0100x;
         $is_j_instr = $instr[6:2] == 5'b11011;
         
         //Decoding Immediate
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}} , $instr[30:20] } :
                   $is_s_instr ? { {21{$instr[31]}} , $instr[30:25] , $instr[11:8] , $instr[7] } :
                   $is_b_instr ? { {20{$instr[31]}} , $instr[7] , $instr[30:25] , $instr[11:8] , 1'b0} :
                   $is_u_instr ? { $instr[31] , $instr[30:20] , $instr[19:12] , 12'b0} : 
                   $is_j_instr ? { {12{$instr[31]}} , $instr[19:12] , $instr[20] , $instr[30:21] , 1'b0} : 32'b0;
         
         //Other fields of Instruction with a when condition
         $rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         $rs1_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         $rd_valid = $is_r_instr || $is_i_instr || $is_u_instr || $is_j_instr;
         $funct7_valid = $is_r_instr;
         $funct3_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
         $opcode[6:0] = $instr[6:0];
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
            
         //decode individual instructions
         $dec_bits[10:0] = {$funct7[5],$funct3,$opcode};
         $is_beq = $dec_bits ==? 11'bx_000_1100011;
         $is_bne = $dec_bits ==? 11'bx_001_1100011;
         $is_blt = $dec_bits ==? 11'bx_100_1100011;
         $is_bge = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu = $dec_bits ==? 11'bx_111_1100011;
         $is_addi = $dec_bits ==? 11'bx_000_0010011;
         $is_add = $dec_bits ==? 11'b0_000_0110011;
         
         
         //Register File Read
         $rf_rd_en1 = $rs1_valid;
         $rf_rd_en2 = $rs2_valid;
         $rf_rd_index1[4:0] = $rs1;
         $rf_rd_index2[4:0] = $rs2;
         
         $src1_value[31:0] = $rf_rd_data1;
         $src2_value[31:0] = $rf_rd_data2;
         
         //ALU operations for ADDI and ADD
          $result[31:0] = $is_addi ? $src1_value + $imm :
                         $is_add ? $src1_value + $src2_value :
                         32'bx;
         
         //Register File Write
         $rf_wr_en =  $rd_valid && ($rd != 5'b0);
         $rf_wr_index[4:0] = $rd;
         $rf_wr_data[31:0] = $result;
         
         //Branch Instructions
         $taken_br = $is_beq ? ($src1_value == $src2_value) :
            $is_bne ? ($src1_value != $src2_value) :
            $is_blt ? (($src1_value < $src2_value)^($src1_value[31] != $src2_value[31])) :
            $is_bge ? (($src1_value >= $src2_value)^($src1_value[31] != $src2_value[31])) :
            $is_bltu ? ($src1_value < $src2_value) :
            $is_bgeu ? ($src1_value >= $src2_value) :
            1'b0;
         
         $br_tgt_pc[31:0] = $pc + $imm;
    
         `BOGUS_USE($is_beq $is_bne $is_blt $is_bge $is_bltu $is_bgeu $is_addi $is_add)
      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   //*passed = *cyc_cnt > 40;
   *passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9);
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

      //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%2010.png)

![Untitled](Day-4%20RISC%20V%20Microarchitecture%2040be8d1cdf1f4069ae21b51ef88e2100/Untitled%2011.png)


# Day-5 Complete Pipelined RISC-V CPU micro-architecture

# Complete RISC-V CPU

Simulation passed!

![Untitled](Day-5%20Complete%20Pipelined%20RISC-V%20CPU%20micro-architec%20d84dd0c4334a4724bfdb940da5cf7195/Untitled.png)

## Code:

```verilog
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   //  r15 (a5): stored/loaded Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   // Store/Load
   m4_asm(SW, r0, r10, 100)             // Store final result in data memory at address 'b100 (4)
   m4_asm(LW, r15, r0, 100)             // Load final result into a5
   // Optional:
   m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;
         $start = !$reset & >>1$reset;
         
         // program counter
         $pc[31:0] = >>1$reset ? 0 :
                     >>3$valid_taken_br ? >>3$br_tgt_pc :
                     >>3$valid_jump     ? >>3$jalr_tgt_pc :
                     >>3$valid_load     ? >>3$pc + 4 :
                     >>1$pc + 4;
         
         // instruction memory read inputs
         $imem_rd_en = !$reset;
         $imem_rd_addr[M4_IMEM_INDEX_CNT-1:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
         
      @1
         // instruction
         $instr[31:0] = $imem_rd_data[31:0];
         
         // instruction type decode (I, R, S, B, J, U)
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] ==  5'b11001;
         
         $is_r_instr = $instr[6:2] ==  5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 ||
                       $instr[6:2] ==  5'b10100;
         
         $is_s_instr = $instr[6:2] ==? 5'b0100x;
         
         $is_b_instr = $instr[6:2] ==  5'b11000;
         
         $is_j_instr = $instr[6:2] ==  5'b11011;
         
         $is_u_instr = $instr[6:2] ==? 5'b0x101;
         
         // instruction decode
         $imm[31:0] = $is_i_instr ? {{21{$instr[31]}}, $instr[30:20]} :
                      $is_s_instr ? {{21{$instr[31]}}, $instr[30:25], $instr[11:7]} :
                      $is_b_instr ? {{20{$instr[31]}}, $instr[7], $instr[30:25], $instr[11:8], 1'b0} :
                      $is_u_instr ? {$instr[31:12], 12'b0} :
                      $is_j_instr ? {{12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:25], $instr[24:21], 1'b0} :
                                    0;
         
         $funct7_valid = $is_r_instr;
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
         
         $rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];
         
         $rs1_valid = $is_i_instr || $is_r_instr || $is_s_instr || $is_b_instr;
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         
         $funct3_valid = $is_i_instr || $is_r_instr || $is_s_instr || $is_b_instr;
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
         
         $rd_valid = $is_i_instr || $is_r_instr || $is_u_instr || $is_j_instr;
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
         
         $opcode[6:0] = $instr[6:0];
         
         // individual instruction decode
         $dec_bits[10:0] = {$funct7[5], $funct3, $opcode};
         
         $is_lui   = $opcode   ==         7'b0110111;
         $is_auipc = $opcode   ==         7'b0010111;
         $is_jal   = $opcode   ==         7'b1101111;
         $is_jalr  = $dec_bits ==? 11'bx_000_1100111;
         $is_beq   = $dec_bits ==? 11'bx_000_1100011;
         $is_bne   = $dec_bits ==? 11'bx_001_1100011;
         $is_blt   = $dec_bits ==? 11'bx_100_1100011;
         $is_bge   = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu  = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu  = $dec_bits ==? 11'bx_111_1100011;
         $is_load  = $opcode   ==         7'b0000011;
         $is_sb    = $dec_bits ==? 11'bx_000_0100011;
         $is_sh    = $dec_bits ==? 11'bx_001_0100011;
         $is_sw    = $dec_bits ==? 11'bx_010_0100011;
         $is_addi  = $dec_bits ==? 11'bx_000_0010011;
         $is_slti  = $dec_bits ==? 11'bx_010_0010011;
         $is_sltiu = $dec_bits ==? 11'bx_011_0010011;
         $is_xori  = $dec_bits ==? 11'bx_100_0010011;
         $is_ori   = $dec_bits ==? 11'bx_110_0010011;
         $is_andi  = $dec_bits ==? 11'bx_111_0010011;
         $is_slli  = $dec_bits ==  11'b0_001_0010011;
         $is_srli  = $dec_bits ==  11'b0_101_0010011;
         $is_srai  = $dec_bits ==  11'b1_101_0010011;
         $is_add   = $dec_bits ==  11'b0_000_0110011;
         $is_sub   = $dec_bits ==  11'b1_000_0110011;
         $is_sll   = $dec_bits ==  11'b0_001_0110011;
         $is_slt   = $dec_bits ==  11'b0_010_0110011;
         $is_sltu  = $dec_bits ==  11'b0_011_0110011;
         $is_xor   = $dec_bits ==  11'b0_100_0110011;
         $is_srl   = $dec_bits ==  11'b0_101_0110011;
         $is_sra   = $dec_bits ==  11'b1_101_0110011;
         $is_or    = $dec_bits ==  11'b0_110_0110011;
         $is_and   = $dec_bits ==  11'b0_111_0110011;
         
      @2
         // register file read
         $rf_rd_en1 = $rs1_valid;
         $rf_rd_index1[4:0] = $rs1;
         $rf_rd_en2 = $rs2_valid;
         $rf_rd_index2[4:0] = $rs2;
         $src1_value[31:0] = (>>1$rf_wr_en && $rs1 == >>1$rd) ? >>1$result : $rf_rd_data1;
         $src2_value[31:0] = (>>1$rf_wr_en && $rs2 == >>1$rd) ? >>1$result : $rf_rd_data2;
         
         // branch and jump instructions
         $taken_br = $is_jal  ? 1'b1 :
                     $is_beq  ? ($src1_value == $src2_value) :
                     $is_bne  ? ($src1_value != $src2_value) :
                     $is_blt  ? ($src1_value <  $src2_value) ^ ($src1_value[31] != $src2_value[31]) :
                     $is_bge  ? ($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31]) :
                     $is_bltu ? ($src1_value <  $src2_value) :
                     $is_bgeu ? ($src1_value >= $src2_value) :
                                1'b0;
         $is_jump = $is_jal || $is_jalr;
         
      @3
         // valid based on previous 2 instructions
         $valid = $reset ? 1'b0 :
                  $start ? 1'b1 :
                           !(>>2$valid_taken_br || >>1$valid_taken_br || >>2$valid_jump || >>1$valid_jump || >>2$valid_load || >>1$valid_load);
         
         // branch and jump valid and pc
         $valid_taken_br = $valid && $taken_br;
         $br_tgt_pc[31:0] = $pc + $imm;
         $valid_jump = $valid && $is_jump;
         $jalr_tgt_pc[31:0] = $src1_value + $imm;
         
         // data memory load valid
         $valid_load = $valid && $is_load;
         
         // ALU
         $sltiu_result[31:0] = $src1_value < $imm;
         $sltu_result[31:0]  = $src1_value < $src2_value;
         $result[31:0] = $is_lui   ? {$imm[31:12], 12'b0} :
                         $is_auipc ? $pc + $imm :
                         $is_jal   ? $pc + 4 :
                         $is_jalr  ? $pc + 4 :
                         $is_load || $is_s_instr ? $src1_value + $imm :
                         $is_addi  ? $src1_value + $imm :
                         $is_slti  ? ($src1_value[31] == $imm[31] ? $sltiu_result : {31'b0, $src1_value[31]}) :
                         $is_sltiu ? $sltiu_result :
                         $is_xori  ? $src1_value ^ $imm :
                         $is_ori   ? $src1_value | $imm :
                         $is_andi  ? $src1_value & $imm :
                         $is_slli  ? $src1_value << $imm[5:0] :
                         $is_srli  ? $src1_value >> $imm[5:0] :
                         $is_srai  ? {{32{$src1_value[31]}}, $src1_value} >> $imm[4:0] :
                         $is_add   ? $src1_value + $src2_value :
                         $is_sub   ? $src1_value - $src2_value :
                         $is_sll   ? $src1_value << $src2_value[4:0] :
                         $is_slt   ? ($src1_value[31] == $src2_value[31] ? $sltu_result : {31'b0, $src1_value[31]}) :
                         $is_sltu  ? $sltu_result :
                         $is_xor   ? $src1_value ^ $src2_value :
                         $is_srl   ? $src1_value >> $src2_value[4:0] :
                         $is_sra   ? {{32{$src1_value[31]}}, $src1_value} >> $src2_value[4:0] :
                         $is_or    ? $src1_value | $src2_value :
                         $is_and   ? $src1_value & $src2_value :
                         32'bx;
         
         // register file write
         $rf_wr_en = (>>2$valid_load) || ($valid && $rd_valid && ($rd != 5'b0) && !$is_load);
         $rf_wr_index[4:0] = $valid ? $rd     : >>2$rd;
         $rf_wr_data[31:0] = $valid ? $result : >>2$ld_data;
         
         // data memory inputs
         $dmem_addr[3:0] = $result[5:2];
         $dmem_wr_en = $is_s_instr;
         $dmem_wr_data[31:0] = $src2_value;
         $dmem_rd_en = $valid_load;
         $dmem_rd_index[5:0] = $src2_value[5:0];
         `BOGUS_USE($dmem_rd_index)
         
      @5
         // data memory output
         $ld_data[31:0] = $dmem_rd_data;
         
      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.
   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = |cpu/xreg[15]>>15$value == (1+2+3+4+5+6+7+8+9);
   *failed = *cyc_cnt > 100;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      m4+rf(@2, @3)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)
   
   m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic
                       // @4 would work for all labs
\SV
   endmodule
```

Link to the project: 

[Makerchip](https://myth.makerchip.com/sandbox/073fmhWJj/0k5hx3#)

## Visualization

On 56th cycle, the reg 10 has the value x2D or d45, which gets stored in to reg 15 after 4-clock cycles.

![Untitled](Day-5%20Complete%20Pipelined%20RISC-V%20CPU%20micro-architec%20d84dd0c4334a4724bfdb940da5cf7195/Untitled%201.png)

![Untitled](Day-5%20Complete%20Pipelined%20RISC-V%20CPU%20micro-architec%20d84dd0c4334a4724bfdb940da5cf7195/Untitled%202.png)

## Diagram

![Untitled](Day-5%20Complete%20Pipelined%20RISC-V%20CPU%20micro-architec%20d84dd0c4334a4724bfdb940da5cf7195/Untitled%203.png)

## Waveforms

![Untitled](Day-5%20Complete%20Pipelined%20RISC-V%20CPU%20micro-architec%20d84dd0c4334a4724bfdb940da5cf7195/Untitled%204.png)

![Untitled](Day-5%20Complete%20Pipelined%20RISC-V%20CPU%20micro-architec%20d84dd0c4334a4724bfdb940da5cf7195/Untitled%205.png)