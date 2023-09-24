# Day-3 Digital Logic with TL-Verilog and Makerchip

Related Course(s): RISC-V Myth (https://www.notion.so/RISC-V-Myth-e52fd09cf2c142c9881dc05a0b7003a2?pvs=21)
Date Last Edited: September 24, 2023 10:21 PM

## Ex. 1 - Validity Tutorial

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled.png)

# Combinational Logic

1. Inverter:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%201.png)

1. 2-input NAND :
    
    ![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%202.png)
    
2. 2-input NOR:
    
    ![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%203.png)
    
3. 2-input XOR:
    
    ![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%204.png)
    

## Ex. 3 -  Vectors:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%205.png)

## Ex. 4 - Mux:

Single bit:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%206.png)

Vector:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%207.png)

## Ex. 5 - Math operators with 4to1 MUX:

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

# Sequential Logic

## Ex. 6 - Free running counter

Logic used:

`$counter[15:0] = $reset ? 0 : (1 + >>1$counter[15:0] );`

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%209.png)

## Ex. 7 - Fibonacchi Series;

Logic used: `$Fibo[31:0] = $reset ? 1 : (>>1$Fibo + >>2$Fibo );`

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2010.png)

## PipeLine Solution:

![Untitled](Day-3%20Digital%20Logic%20with%20TL-Verilog%20and%20Makerchip%2073f82059fc62479497b87e58bdb8ee9a/Untitled%2011.png)

2 cycle calc:

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

2-cycle calc with valid:

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