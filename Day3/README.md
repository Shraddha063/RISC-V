
# Day 3 – Digital Logic Design Using TL-Verilog and Makerchip

## Overview

Day 3 focused on implementing digital logic using Transaction-Level Verilog (TL-Verilog) in the Makerchip IDE. The exercises covered arithmetic operations, sequential logic, pipelining, and pipeline optimization.

---

## 1. Sequential Calculator – Arithmetic Operations

Implemented a sequential calculator supporting four arithmetic operations:

- Addition
- Subtraction
- Multiplication
- Division

### Design Implementation

- Generated randomized input operands.
- Used a 2-bit operation selector.
- Used the previous output as an operand for subsequent calculations.
- Implemented division-by-zero protection.
- Incorporated reset and validity tracking.

### RTL Logic

```verilog
$sum[31:0]  = $val1 + $val2;
$diff[31:0] = $val1 - $val2;
$prod[31:0] = $val1 * $val2;

$quot[31:0] = ($val2 != 0) ?
              ($val1 / $val2) : 32'hDEAD_BEEF;
```

### Block Diagram

![Sequential Calculator Block Diagram](./Screenshots/Screenshot%202026-11-26%20105121.png)

### Simulation Waveform

![Sequential Calculator Waveform](./Screenshots/Screenshot%202026-11-26%20105143.png)

---

## 2. Pipelined Pythagorean Calculation – 4 Stages

Implemented a four-stage pipelined Pythagorean calculation.

**Objective:**

Calculate:

\[
c = \sqrt{a^2+b^2}
\]

### Pipeline Implementation

| Stage | Operation |
|---|---|
| @1 | Calculate square of first input |
| @2 | Calculate square of second input |
| @3 | Add squared values |
| @4 | Calculate square root |

### RTL Logic

```verilog
|calc
    @1
        $aa_sq[31:0] = $aa * $aa;

    @2
        $bb_sq[31:0] = $bb * $bb;

    @3
        $cc_sq[31:0] = $aa_sq + $bb_sq;

    @4
        $out[31:0] = sqrt($cc_sq);
```

### Block Diagram

![Four Stage Pipelined Pythagorean Calculator](./Screenshots/Screenshot%202026-11-26%20111909.png)

### Simulation Waveform

![Four Stage Pipeline Simulation Waveform](./Screenshots/Screenshot%202026-11-26%20112426.png)

---

## 3. Optimized Pipelined Pythagorean Calculation – 3 Stages

Optimized the previous design by calculating both input squares simultaneously in the first pipeline stage.

This reduced the pipeline from four stages to three stages.

### Pipeline Implementation

| Stage | Operation |
|---|---|
| @1 | Calculate squares of both inputs simultaneously |
| @2 | Add squared values |
| @3 | Calculate square root |

### RTL Logic

```verilog
|calc
    @1
        $aa_sq[7:0] = $aa[3:0] ** 2;
        $bb_sq[7:0] = $bb[3:0] ** 2;

    @2
        $cc_sq[8:0] = $aa_sq + $bb_sq;

    @3
        $cc[4:0] = sqrt($cc_sq);
```

### Block Diagram

![Three Stage Optimized Pythagorean Calculator](./Screenshots/Screenshot%202026-11-26%20113402.png)

### Simulation Waveform

![Three Stage Pipeline Simulation Waveform](./Screenshots/Screenshot%202026-11-26%20113531.png)

---
## 4-bit Counter

Designed a resettable 4-bit counter using sequential logic in TL-Verilog. The counter increments on each clock cycle and resets when the reset signal is asserted.

**Counter implementation:**

![4-bit Counter RTL](./Screenshots/Screenshot%202026-11-25%20152402(1).png)

**Simulation waveform:**

![4-bit Counter Waveform](./Screenshots/Screenshot%202026-11-25%20152417(1).png)

## Fibonacci Sequence Generator

Implemented a Fibonacci sequence generator using sequential logic and pipeline registers. The design uses previous clock-cycle values to calculate the next Fibonacci number.

**Fibonacci implementation:**

![Fibonacci RTL and Diagram](./Screenshots/Screenshot%202026-11-25%20153119(1).png)

**Simulation waveform:**

![Fibonacci Waveform](./Screenshots/Screenshot%202026-11-25%20153136(1).png)


## Sequential Calculator

Implemented a sequential calculator supporting arithmetic operations such as addition, subtraction, multiplication, and division. The design uses registered outputs and handles division-by-zero conditions.

**Sequential calculator implementation:**

![Sequential Calculator RTL and Diagram](./Screenshots/Screenshot%202026-11-25%20164844(1).png)

**Simulation waveform:**

![Sequential Calculator Waveform](./Screenshots/Screenshot%202026-11-25%20165017(1).png)


## Pythagorean Calculation Using Pipelining

Implemented a Pythagorean calculation using pipelined stages. The design calculates the squares of the input values, adds them, and computes the square root of the result.

**Pipelined implementation:**

![Pythagorean Pipeline RTL and Diagram](./Screenshots/Screenshot%202026-11-25%20161103(1).png)

**Simulation waveform:**

![Pythagorean Pipeline Waveform](./Screenshots/Screenshot%202026-11-25%20163715(1).png)

## Key Learnings

- Implemented sequential arithmetic operations using TL-Verilog.
- Explored arithmetic operators and conditional expressions.
- Implemented division-by-zero protection.
- Understood pipeline stages and timing abstractions.
- Explored parallel execution of independent operations.
- Optimized pipeline organization from four stages to three stages.
- Analyzed simulation waveforms and signal propagation.

## Tools Used

- **HDL:** Transaction-Level Verilog (TL-Verilog)
- **IDE:** Makerchip
- **Simulation:** Makerchip Waveform Viewer

---

**Day 3 Completed – Sequential Logic, Arithmetic Operations, and Pipeline Optimization using TL-Verilog.**
