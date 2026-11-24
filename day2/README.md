
# Day 2 – Introduction to ABI and Basic Verification Flow

## Overview

Day 2 of the RISC-V MYTH workshop focuses on understanding the RISC-V Application Binary Interface (ABI), calling conventions, C-to-assembly interaction, debugging using Spike, and running a RISC-V program on a Verilog-based CPU using Icarus Verilog.

### Objectives

- Understand the RISC-V ABI and function calling conventions.
- Understand how C programs call assembly functions.
- Write and compile RISC-V assembly code.
- Inspect compiled programs using `objdump`.
- Execute and debug programs using Spike.
- Run a RISC-V firmware program on the PicoRV32 processor.
- Understand the basic RTL simulation flow using Icarus Verilog.

---

## 1. Workspace Setup and Required Files

Created a separate working directory for Day 2:

```bash
mkdir -p ~/riscv_day2
cd ~/riscv_day2
```

Copied the required workshop files:

```bash
cp ~/riscv_workshop_collaterals/labs/{syscalls.c,riscv.ld,start.S,start.ld,hex8tohex32.py,picorv32.v,testbench.v} .
```

The workspace contains the following important files:

| File | Description |
|---|---|
| `1to9_custom.c` | C program that calls an assembly function |
| `load.S` | RISC-V assembly implementation of the function |
| `start.S` | Startup assembly code |
| `syscalls.c` | System-call support |
| `riscv.ld` | Linker script |
| `start.ld` | Startup linker script |
| `picorv32.v` | PicoRV32 processor RTL |
| `testbench.v` | Verilog testbench |
| `hex8tohex32.py` | Converts firmware hex data |
| `rv32im.sh` | Compilation and simulation script |

### Workspace and file verification

![Day 2 workspace](Screenshots/Screenshot%202026-11-24%20105157.png)

---

## 2. Understanding the RISC-V ABI

The Application Binary Interface (ABI) defines how software components interact with one another.

It specifies:
- How function arguments are passed.
- How return values are handled.
- Which registers must be preserved.
- How the stack is managed.
- How functions call and return from other functions.

### Important RISC-V argument and return registers

| Register | Purpose |
|---|---|
| `a0–a7` | Function arguments |
| `a0–a1` | Return values |
| `ra` | Return address |
| `sp` | Stack pointer |
| `zero` | Constant zero |
| `s0–s11` | Callee-saved registers |
| `t0–t6` | Temporary registers |

The first two function arguments are passed through `a0` and `a1`. The return value is generally placed in `a0`.

---

## 3. Task 1 – Calling an Assembly Function from C

### 3.1 C program

Created `1to9_custom.c`.

The C program calls an externally defined assembly function named `load()`.

```c
#include <stdio.h>

extern int load(int x, int y);

int main()
{
    int result = 0;
    int count = 9;

    result = load(0, count + 1);

    printf("Sum of numbers from 1 to %d is %d\n",
           count, result);

    return 0;
}
```

The function call is:

```c
result = load(0, count + 1);
```

The arguments passed are:
- `a0 = 0`
- `a1 = 10`

![C program](Screenshots/Screenshot%202026-11-24%20105235.png)

### 3.2 Assembly implementation

Created `load.S` to implement the `load()` function.

```asm
.section .text
.global load
.type load, @function

load:
    add  a4, a0, zero
    add  a2, a0, a1
    add  a3, a0, zero

loop:
    add  a4, a3, a4
    addi a3, a3, 1
    blt  a3, a2, loop

    add  a0, a4, zero
    ret
```

### 3.3 Understanding the assembly instructions

| Instruction | Operation |
|---|---|
| `add` | Addition |
| `addi` | Addition with immediate |
| `blt` | Branch if less than |
| `ret` | Return from function |

The assembly function initializes the accumulator and counter, iterates through the loop, and returns the calculated result through `a0`.

The loop adds the values from 0 to 9. Since adding zero does not change the sum, the result is 45.

![Assembly implementation](Screenshots/Screenshot%202026-11-24%20105251.png)

### 3.4 Understanding the function call in disassembly

The generated machine instructions can be inspected using:

```bash
riscv64-unknown-elf-objdump -d 1to9_custom.o
```

The disassembly helps verify how the compiler passes arguments and calls the assembly function.

The `jal` instruction transfers control to `load`, while saving the return address in `ra`.

![Disassembly of C and assembly program](Screenshots/Screenshot%202026-11-24%20105304.png)

Additional disassembly inspection:

![Disassembly output](Screenshots/Screenshot%202026-11-24%20105326.png)

### 3.5 Executing using Spike

Executed the compiled program using the RISC-V Spike simulator:

```bash
spike pk 1to9_custom.o
```

**Output:**

```text
bbl loader
Sum of numbers from 1 to 9 is 45
```

![Spike execution](Screenshots/Screenshot%202026-11-24%20105708.png)

---

## 4. Task 2 – Running the Program on PicoRV32 RTL

The next part involves running the C and assembly program on the PicoRV32 processor using Verilog simulation.

### 4.1 PicoRV32 processor

`picorv32.v` contains the RTL implementation of the PicoRV32 RISC-V processor.

The testbench instantiates the processor and provides:
- Clock generation.
- Reset generation.
- Instruction and data memory interface.
- Firmware loading.
- Memory-mapped output.
- Simulation termination.

### 4.2 Testbench

The processor is instantiated in `testbench.v`.

```verilog
picorv32 #(
    .COMPRESSED_ISA(1),
    .ENABLE_MUL(1),
    .ENABLE_DIV(1)
) uut (
    .clk(clk),
    .resetn(resetn),
    .trap(trap),
    .mem_valid(mem_valid),
    .mem_instr(mem_instr),
    .mem_ready(mem_ready),
    .mem_addr(mem_addr),
    .mem_wdata(mem_wdata),
    .mem_wstrb(mem_wstrb),
    .mem_rdata(mem_rdata)
);
```

The processor configuration enables:
- Compressed instruction support.
- Multiplication support.
- Division support.

The memory is initialized using `firmware32.hex`.

![PicoRV32 testbench and processor instantiation](Screenshots/Screenshot%202026-11-24%20111802.png)

### 4.3 Memory-mapped output

The testbench uses a memory-mapped address to print characters generated by the firmware.

```verilog
if (mem_addr == 32'h1000_0000) begin
    $write("%c", mem_wdata[7:0]);
end
```

This allows the simulated processor to display the program output in the terminal.

![Memory-mapped output implementation](Screenshots/Screenshot%202026-11-24%20112805.png)

### 4.4 Compilation and simulation flow

The workshop script `rv32im.sh` performs the following operations:

1. Compiles the C program for RV32IM.
2. Compiles the assembly function.
3. Compiles the system-call support code.
4. Links the firmware.
5. Generates the startup ELF.
6. Converts ELF files into Verilog hex format.
7. Converts the firmware hex data into 32-bit format.
8. Compiles the PicoRV32 RTL and testbench using Icarus Verilog.
9. Runs the simulation using `vvp`.

Execute the script:

```bash
bash ~/riscv_workshop_collaterals/labs/rv32im.sh
```

![Simulation script and execution](Screenshots/Screenshot%202026-11-24%20113632.png)

### 4.5 Firmware generation and hex conversion

The script generates:

- `firmware.elf`
- `start.elf`
- `firmware.hex`
- `firmware32.hex`

The hex conversion is necessary to initialize the memory in the Verilog testbench.

![Firmware generation and conversion](Screenshots/Screenshot%202026-11-24%20113658.png)

![Hex conversion and simulation preparation](Screenshots/Screenshot%202026-11-24%20113723.png)

![Generated firmware files](Screenshots/Screenshot%202026-11-24%20113732.png)

---

## 5. Troubleshooting – PicoRV32 TRAP During Simulation

### 5.1 Initial simulation issue

During the first simulation attempt, the terminal displayed:

```text
TRAP
```

The expected C program output was missing.

To identify the cause, the trap handling section in `testbench.v` was modified to print the program counter when a trap occurred.

```verilog
$display("TRAP at PC = %h", uut.reg_pc);
```

The simulation showed:

```text
TRAP at PC = 0001fae8
TRAP
```

![Initial trap during simulation](Screenshots/Screenshot%202026-11-24%20113632.png)

### 5.2 Identifying the cause

The instruction at address `0x1FAE8` was inspected using:

```bash
riscv64-unknown-elf-objdump -d firmware.elf
```

The disassembly showed instructions such as:

```asm
remu
divu
```

These instructions belong to the RISC-V M extension.

The PicoRV32 core had multiplication and division disabled by default:

```verilog
parameter [0:0] ENABLE_MUL = 0;
parameter [0:0] ENABLE_DIV = 0;
```

Therefore, the processor could not execute the division and remainder instructions used by the compiled firmware.

![Disassembly showing M-extension instructions](Screenshots/Screenshot%202026-11-24%20113723.png)

### 5.3 Fix applied

Enabled the multiplication and division units in the PicoRV32 instance inside `testbench.v`.

Changed:

```verilog
picorv32 #(
    .COMPRESSED_ISA(1)
) uut (
```

To:

```verilog
picorv32 #(
    .COMPRESSED_ISA(1),
    .ENABLE_MUL(1),
    .ENABLE_DIV(1)
) uut (
```

This enables the required M-extension operations.

![Updated PicoRV32 configuration](Screenshots/Screenshot%202026-11-24%20113732.png)

### 5.4 Successful simulation

Re-ran the simulation:

```bash
bash ~/riscv_workshop_collaterals/labs/rv32im.sh
```

**Output:**

```text
Sum of numbers from 1 to 9 is 45
TRAP at PC = 000102cc
TRAP
```

The expected result was printed successfully.

![Successful PicoRV32 simulation output](Screenshots/Screenshot%202026-11-24%20121611.png)

### 5.5 Understanding the final TRAP

The final trap was investigated by disassembling the firmware around address `0x102CC`:

```bash
riscv64-unknown-elf-objdump -d firmware.elf | grep -B8 -A12 '102cc'
```

The disassembly showed:

```asm
000102bc <_exit>:
    addi sp,sp,-32
    sw   s0,28(sp)
    addi s0,sp,32
    sw   a0,-20(s0)
    ebreak
```

The `ebreak` instruction is part of the `_exit` routine. The program reaches this instruction after returning from `main()`.

The testbench detects the processor's trap signal and terminates the simulation.

**Conclusion:** The final `TRAP` is the expected termination mechanism in this simulation flow. The program successfully prints the expected result before termination.

![Final trap address and ebreak instruction](Screenshots/Screenshot%202026-11-24%20113723.png)

---

## 6. Complete Simulation Flow

```text
C Program + RISC-V Assembly
          |
          v
RV32IM Compilation and Linking
          |
          v
      firmware.elf
          |
          v
      Hex Conversion
          |
          v
     firmware32.hex
          |
          v
 Icarus Verilog Compilation
          |
          v
 PicoRV32 RTL + Testbench
          |
          v
       Simulation
          |
          v
Expected Output: 45
          |
          v
  EBREAK / TRAP Termination
```

---

## 7. Key Learnings

- Understood the RISC-V ABI and function calling conventions.
- Learned how C programs interact with assembly functions.
- Understood argument passing and return values through RISC-V registers.
- Inspected compiled machine code using `objdump`.
- Executed programs using Spike.
- Understood how firmware is compiled, linked, and converted into memory initialization files.
- Learned how a Verilog testbench interacts with a RISC-V processor.
- Understood memory-mapped output in RTL simulation.
- Debugged an unsupported M-extension instruction issue in PicoRV32.
- Learned how to enable multiplication and division support in the processor configuration.
- Verified successful execution of the firmware on the PicoRV32 RTL model.

---

## 8. Tools Used

| Tool | Purpose |
|---|---|
| RISC-V GCC | Compilation |
| RISC-V objdump | Disassembly |
| Spike | RISC-V instruction-set simulation |
| Icarus Verilog | RTL compilation and simulation |
| VVP | Running the compiled simulation |
| PicoRV32 | RISC-V processor RTL |
| Ubuntu | Development environment |
| Verilog | Testbench and RTL simulation |

---

## Day 2 – Conclusion

Successfully completed the C-to-assembly function call exercise and executed the program using Spike.

Also completed the basic RTL verification flow by compiling the firmware, loading it into the PicoRV32 testbench, debugging the M-extension configuration issue, and verifying the expected output:

```text
Sum of numbers from 1 to 9 is 45
```

The final `ebreak`-based trap was confirmed as the simulation's normal termination mechanism.
