# Day-5 Complete Pipelined RISC-V CPU micro-architecture

Related Course(s): RISC-V Myth (https://www.notion.so/RISC-V-Myth-e52fd09cf2c142c9881dc05a0b7003a2?pvs=21)
Date Last Edited: September 26, 2023 7:46 PM

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