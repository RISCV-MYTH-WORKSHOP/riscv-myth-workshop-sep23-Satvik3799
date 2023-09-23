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
