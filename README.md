# Assignment3: Single-cycle RISC-V CPU
Contirbuted by < [Brian Cheng](https://github.com/BrianCheng-TheLegend) >

## [Lab3: Construct a single-cycle RISC-V CPU with Chisel](https://hackmd.io/@sysprog/r1mlr3I7p) 
* Single-cycle CPU(MyCPU) Structure
![image](https://hackmd.io/_uploads/SJx9txvra.png)
### CPU Test
This is my [repository](https://github.com/BrianCheng-TheLegend/ca2023-lab3) with completed MyCPU.
```
sbt test
[info] welcome to sbt 1.9.7 (Oracle Corporation Java 17.0.6)
[info] loading settings for project ca2023-lab3-build from plugins.sbt ...
[info] loading project definition from /home/player1/Desktop/ca2023-lab3/projec
[info] loading settings for project root from build.sbt ...
[info] set current project to mycpu (in build file:/home/player1/Desktop/ca2023-lab3/)
[info] ExecuteTest:
[info] Execution of Single Cycle CPU
[info] - should execute correctly
[info] InstructionDecoderTest:
[info] InstructionDecoder of Single Cycle CPU
[info] - should produce correct control signal
[info] QuicksortTest:
[info] Single Cycle CPU
[info] - should perform a quicksort on 10 numbers
[info] InstructionFetchTest:
[info] InstructionFetch of Single Cycle CPU
[info] - should fetch instruction
[info] ByteAccessTest:
[info] Single Cycle CPU
[info] - should store and load a single byte
[info] FibonacciTest:
[info] Single Cycle CPU
[info] - should recursively calculate Fibonacci(10)
[info] RegisterFileTest:
[info] Register File of Single Cycle CPU
[info] - should read the written content
[info] - should x0 always be zero
[info] - should read the writing content
[info] Run completed in 9 seconds, 291 milliseconds.
[info] Total number of tests run: 9
[info] Suites: completed 7, aborted 0
[info] Tests: succeeded 9, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 10 s, completed Dec 1, 2023, 6:41:54 PM

```
### Modified
1. InstructionFetch.scala
2. InstructionDecoder.scala
3. Ececute.scala
4. Cpu.scala



### Instruction Fetch 
[InstructionFetch.scala](https://github.com/BrianCheng-TheLegend/ca2023-lab3/blob/main/src/main/scala/riscv/core/InstructionFetch.scala)
![image](https://hackmd.io/_uploads/S1pfI8wS6.png =50%x)
Instruction Fetch is the initial step in the instruction cycle of a computer's central processing unit (CPU). 
In this stage, the position of the program executed in the previous cycle will be received, along with whether it will jump to another code location and the destination of the jump if applicable.

* **Test `InstructionFetch_of_Single_Cycle_CPU_should_fetch_instruction`**
The test will check the `io_jump_flag` will or will not chang between `1` and `0` when the jump has happened.

    * Waveform
First time when `clock` signal is 1 we can see the `instruction_address` is `0x1000`.
![image](https://hackmd.io/_uploads/SJrKMwPH6.png)
At `clock` signal is 0 the `instruction_address` won't change.
![image](https://hackmd.io/_uploads/Hy_9GDPBp.png)
When `clock` signal turn to 1 agnain `instruction_address` add 4 to `0x1004`.
![image](https://hackmd.io/_uploads/SJZjMDvrp.png)
And this will match our expect because we will expect this stage will fetch the instruction on tis address and move to next address at next cycle.


### Instruction Decode 
[InstructionDecoder.scala](d/ca2023-lab3/blob/main/src/main/scala/riscv/core/InstructionDecode.scala)
![image](https://hackmd.io/_uploads/H1TP8LDHp.png =50%x)
In this stage, we will receive the program code fetched by the Instruction Fetch. We will then break down this code into opcode, function3, function7, register1, register2, and rd like the picture below.
![image](https://hackmd.io/_uploads/HJH3FLvr6.png =60%x)

* **Test `InstructionDecoder_of_Single_Cycle_CPU_should_produce_correct_control_signal`**
This will test at this stage will or will not perform `S-type`  `lui`  `add` instructure as expect.

    * Waveform
We can see the instructure is `io_instruction = 0xA02223` and we know it is`sw x10, 4(x0)`.
And the `io_memory_write_enable = 1` is match our expect.
![image](https://hackmd.io/_uploads/H1rBTwDrp.png)
We can see the instructure is `io_instruction = 0x22B7` and we know it is`lui x5, 2`.
The `io_reg_write_enable = 1` is match our expect and the `io_reg_write_address = 5` is match the `x5` register.
![image](https://hackmd.io/_uploads/HJJdaPDBT.png)


### Execute  
[Execute.scala](https://github.com/BrianCheng-TheLegend/ca2023-lab3/blob/main/src/main/scala/riscv/core/Execute.scala)
![image](https://hackmd.io/_uploads/B1j58LPrT.png =50%x)
In this stage, we will receive data from the Instruction Decoder. The red line of ALUOp1Src is used to determine whether the ALU should receive data from Register 1 or the instruction address. Similarly, ALUOp2Src is utilized to determine whether the ALU should receive data from Register 2 or the immediate value.

* **Test `Execution_of_Single_Cycle_CPU_should_execute_correctly`**
This stage will test `add` and `beq` instruction when `equ` and `not equ` happened the funtion will perform as expect or not. 

    * Waveform
The instruction in this stage depends on `io_instruction = 0x1101B3` we can know it will be `add x3, x2, x1` and we can see `opcode=0x33` will match the expect with `add`.
![image](https://hackmd.io/_uploads/BJs-yuwrp.png)
This instruction is `io_instruction = 0x208163` and we can see the `io_if_jump_flag = 1`.
![image](https://hackmd.io/_uploads/SkKCJ_Prp.png)


### Register
![image](https://hackmd.io/_uploads/rkBpU8DHp.png =50%x)

* **Test `Register_File_of_Single_Cycle_CPU_should_read_the_written_content`**
In this test we will look for the if the stage can read the written content.
    * Waveform
In this stage you can see the `io_wirte_data = 0xDEADBEEF` and `io_write_enable = 1` when the `io_wirte_address = 1` which`registers_1 = 0` .
![image](https://hackmd.io/_uploads/BJHUBFDHT.png)
The next `clock` the `registers_1 ` changed `registers_1 = 0xDEADBEEF`.
![image](https://hackmd.io/_uploads/H1JDBFvBT.png)


* **Test `Register_File_of_Single_Cycle_CPU_should_x0_always_be_zero`**
In this test we will look for if the register x0 be written will still be 0.
    * Waveform
In this stage you can see the `io_wirte_data = 0xDEADBEEF` and `io_write_enable = 1` when `io_wirte_address = 1` which point at `registers_0 = 0`.
![image](https://hackmd.io/_uploads/Syf1DKvHT.png)
After last `clock` we can see the `registers_0 = 0` still remain 0 and this will match our expect.
![image](https://hackmd.io/_uploads/SJh1PFwrT.png)


* **Test `Register_File_of_Single_Cycle_CPU_should_read_the_writing_content`**
In this test we will look for the if the stage can read the writting content.
    * Waveform
In this stage we can see the `io_wirte_data = 0xDEADBEEF` and the `io_wirte_address = 2` and we can notice that `registers_2 = 0` the value of this registers still not change.
![image](https://hackmd.io/_uploads/S1UA7YDBT.png)
And at next `clock` we can notice that`registers_2 = 0` change to `0xDEADBEEF` and at the same `clock` the value of `io_read_data1 = 0xDEADBEEF`, the value read and write at the same time.
![image](https://hackmd.io/_uploads/ryC6mKwB6.png)
The test is match our expect.


### CPU 
[CPU.scala](https://github.com/BrianCheng-TheLegend/ca2023-lab3/blob/main/src/main/scala/riscv/core/CPU.scala)
There are three test in [CPU.scala](https://github.com/BrianCheng-TheLegend/ca2023-lab3/blob/main/src/test/scala/riscv/singlecycle/CPUTest.scala), and I will choose `quicksort` to illustrate it
* **Test `Single_Cycle_CPU_should_perform_a_quicksort_on_10_numbers`**
    * Waveform
:::warning
Ongoing
:::

### Memory Access
![image](https://hackmd.io/_uploads/BylkvLvHa.png)

### Write back
![image](https://hackmd.io/_uploads/HknewLDBT.png)


## Implement [Hw2](https://hackmd.io/@PWCheng/CAHW02) Code to MyCPU
Modify the code to to fit MyCPU.
:::spoiler Modified Assembly code 
```
.org 0
.global _start

/* newlib system calls */
.set STDOUT,1
.set SYSEXIT, 93
.set SYSWRITE, 64

.data
    # will not overflow, and will predict as false
cmp_data_1: .dword 0x0000000000000000, 0x0000000000000000
    # will not overflow, and will predict as false
cmp_data_2: .dword 0x0000000000000001, 0x0000000000000010
    # will not overflow, but will predict as true
cmp_data_3: .dword 0x0000000000000002, 0x4000000000000000
    # will overflow, and will predict as true
cmp_data_4: .dword 0x0000000000000003, 0x7FFFFFFFFFFFFFFF

nextline:    .ascii  "\n"
             .set str_next_len, .-nextline

blank:      .ascii  " "
             .set str_blank_len, .-blank



.text
# assume little endian
_start:
    addi sp, sp, -16
    
    # push four pointers of test data onto the stack
    la t0, cmp_data_1
    sw t0, 0(sp)
    la t0, cmp_data_2
    sw t0, 4(sp)
    la t0, cmp_data_3
    sw t0, 8(sp)
    la t0, cmp_data_4
    sw t0, 12(sp)
 
    addi s0, zero, 4    # s0 is the goal iteration count
    addi s1, zero, 0    # s1 is the counter
    addi s2, sp, 0      # s2 now points to cmp_data_1
main_loop:
    lw a0, 0(s2)        # a0 stores the pointer to first data in cmp_data_x
    addi a1, a0, 8      # a1 stores the pointer to second data in cmp_data_x
    jal ra, pimo

    ### print for rv32emu
    addi a1, a0, 48
    addi sp, sp, -4
    sw a1, 0(sp)
    addi a1, sp, 0
    li a7, SYSWRITE
    li a0, STDOUT
    li a2, 4
    # ecall 
    addi sp,sp,4  
    ###

    # printf("\n");
    li  a7, SYSWRITE
    li  a0, 1
    la  a1, nextline
    li  a2, 1
    # ecall
    
    addi s2, s2, 4      # s2 points to next cmp_data_x
    addi s1, s1, 1      # counter++
    bne s1, s0, main_loop
    
    addi sp, sp, 16
    j exit
    
    
# predict if multiplication overflow:
pimo:
    addi sp, sp, -20
    sw ra, 0(sp)
    sw s0, 4(sp)
    sw s1, 8(sp)
    sw s2, 12(sp)
    sw s3, 16(sp)
    
    mv s0, a0       # s0 is address of x0
    mv s1, a1       # s1 is address of x1
    
    lw a0, 0(s0)
    lw a1, 4(s0)    # a0 a1 is now the value of x0
    jal ra, clz
    li s2, 63
    sub s2, s2, a0  # s2 is now exp_x0
    
    lw a0, 0(s1)
    lw a1, 4(s1)    # a1 a0 is now the value of x1
    jal ra, clz
    li s3, 63
    sub s3, s3, a0  # s3 is now exp_x1
 
    add s2, s2, s3
    addi s2, s2, 2  # s2 is (exp_x0 + 1) + (exp_x1 + 1)
    li t0, 64
    bge s2, t0, pimo_ret_t
    li a0, 0        # return false
    j pimo_end
pimo_ret_t:
    li a0, 1        # return true
pimo_end:
    lw ra, 0(sp)
    lw s0, 4(sp)
    lw s1, 8(sp)
    lw s2, 12(sp)
    lw s3, 16(sp)
    addi sp, sp, 20
    ret


# count leading zeros
clz:
    addi sp, sp, -4
    sw ra, 0(sp)
    
    # a0 a1 = x

    bne a1, zero, clz_fill_ones_upper
clz_fill_ones_lower:
    srli t0, a0, 1
    or a0, a0, t0
    srli t0, a0, 2
    or a0, a0, t0
    srli t0, a0, 4
    or a0, a0, t0
    srli t0, a0, 8
    or a0, a0, t0
    srli t0, a0, 16
    or a0, a0, t0
    j clz_fill_ones_end
clz_fill_ones_upper:
    srli t1, a1, 1
    or a1, a1, t1
    srli t1, a1, 2
    or a1, a1, t1
    srli t1, a1, 4
    or a1, a1, t1
    srli t1, a1, 8
    or a1, a1, t1
    srli t1, a1, 16
    or a1, a1, t1
    li a0, 0xffffffff
clz_fill_ones_end:
    
    
    # x -= ((x >> 1) & 0x5555555555555555);
    srli t0, a0, 1
    slli t1, a1, 31
    or t0, t0, t1
    srli t1, a1, 1      # t0 t1 = x >> 1
    
    li t2, 0x55555555   # t2 is the mask
    and t0, t0, t2
    and t1, t1, t2      # t0 t1 = (x >> 1) & 0x5555555555555555
 
    sltu t3, a0, t0     # t3 is the borrow bit
    sub a0, a0, t0
    sub a1, a1, t1
    sub a1, a1, t3      # a0 a1 = x - (t0 t1)
    
    
    # x = ((x >> 2) & 0x3333333333333333) + (x & 0x3333333333333333);
    srli t0, a0, 2
    slli t1, a1, 30
    or t0, t0, t1
    srli t1, a1, 2      # t0 t1 = x >> 2
    
    li t2, 0x33333333   # t2 is the mask
    and t0, t0, t2
    and t1, t1, t2      # t0 t1 = (x >> 2) & 0x3333333333333333
    and t4, a0, t2
    and t5, a1, t2      # t4 t5 = x & 0x3333333333333333
    
    add a0, t0, t4
    sltu t3, a0, t0     # t3 is the carry bit
    add a1, t1, t5
    add a1, a1, t3      # a0 a1 = (t0 t1) + (t4 t5)
    
    
    # x = ((x >> 4) + x) & 0x0f0f0f0f0f0f0f0f;
    srli t0, a0, 4
    slli t1, a1, 28
    or t0, t0, t1
    srli t1, a1, 4      # t0 t1 = x >> 4
    
    add t0, t0, a0
    sltu t3, t0, a0     # t3 is the carry bit
    add t1, t1, a1
    add t1, t1, t3      # t0 t1 = (x >> 4) + x
    
    li t2, 0x0f0f0f0f   # t2 is the mask
    and a0, t0, t2
    and a1, t1, t2      # a0 a1 = (t0 t1) & 0x0f0f0f0f0f0f0f0f
    
    
    # x += (x >> 8);
    srli t0, a0, 8
    slli t1, a1, 24
    or t0, t0, t1
    srli t1, a1, 8      # t0 t1 = x >> 8
    
    add a0, a0, t0
    sltu t3, a0, t0     # t3 is the carry bit
    add a1, a1, t1
    add a1, a1, t3      # a0 a1 = x + (x >> 8)
    
    
    # x += (x >> 16);
    srli t0, a0, 16
    slli t1, a1, 16
    or t0, t0, t1
    srli t1, a1, 16     # t0 t1 = x >> 16
    
    add a0, a0, t0
    sltu t3, a0, t0     # t3 is the carry bit
    add a1, a1, t1
    add a1, a1, t3      # a0 a1 = x + (x >> 16)
    
    
    # x += (x >> 32);
    mv t0, a1
    mv t1, zero         # t0 t1 = x >> 32
    
    add a0, a0, t0
    sltu t3, a0, t0     # t3 is the carry bit
    add a1, a1, t1
    add a1, a1, t3      # a0 a1 = x + (x >> 32)
    
    
    # return (64 - (x & 0x7f));
    andi a0, a0, 0x7f   # a0 = (x & 0x7f)
    li t0, 64
    sub a0, t0, a0      # a0 = (64 - (x & 0x7f))
    
    
    lw ra, 0(sp)
    addi sp, sp, 4
    ret

exit:
    li  a7, SYSEXIT	    
    addi a0, x0, 0
    # ecall
```
:::
Remove `ecall` from original code.

Move [`hw02.S`](https://github.com/BrianCheng-TheLegend/ca2023-lab3/blob/main/csrc/hw02.S) to `./CA2023-LAB3/csrc` and make to create `hw02.asmbin`.

Add the code on `CPUTest.scala`
```scala!
class Hw02Test extends AnyFlatSpec with ChiselScalatestTester {
  behavior.of("Single Cycle CPU")
  it should "Run the Hw02 Test." in {
    test(new TestTopModule("hw02.asmbin")).withAnnotations(TestAnnotations.annos) { c =>
      for (i <- 1 to 500) {
        c.clock.step(1000)
        c.io.mem_debug_read_address.poke((i * 4).U) // Avoid timeout
      }
      c.clock.step()
      c.io.mem_debug_read_data.expect(0.U)
    }
  }
}
```
    The first instruciton is `addi sp, sp, -16` 
* Instruction Fetch
    * Waveform
    ![image](https://hackmd.io/_uploads/BJWPV-JLp.png)
    We can see the `io_instruction_read_data = 0xff010113` and the decode of this instruction will be `addi sp, sp, -16` .
    
* Instruciton Decode
    * Waveform
    ![image](https://hackmd.io/_uploads/SyuiNb1IT.png)


* Execute
    * Waveform
    ![image](https://hackmd.io/_uploads/SkJA4-JUT.png)
    
    
* Memory Access
    * Waveform
    ![image](https://hackmd.io/_uploads/HkayI-1Ip.png)
    
    
* Write Back
    * Waveform
    ![image](https://hackmd.io/_uploads/HJzBwZJ86.png)
    
    
* Registers
    * Waveform
    ![image](https://hackmd.io/_uploads/H1yHu-1L6.png)


* Verilator
    * We use verilator
    
# Reference 
[Lab3: Construct a single-cycle RISC-V CPU with Chisel](https://hackmd.io/@sysprog/r1mlr3I7p)