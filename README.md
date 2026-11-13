Memory-to-memory DMA controller in SystemVerilog with a self-checking testbench and C-model/RTL test reuse, verified with Verilator.
# dma-soc-verification

A small pre-silicon style project: a memory-to-memory DMA controller with an APB register
interface, verified two ways.

- **SystemVerilog testbench:** self-checking, with directed tests, 100 randomized transfers,
  and assertions.
- **C++ test framework:** the same tests run on a C model and on the Verilated RTL, to show
  test reuse across platforms.

## Features
- Registers: SRC, DST, LEN, CTRL, STATUS, CLEAR
- Interrupt output with a write-1-to-clear DONE flag
- Error handling for zero length, unaligned addresses, and out-of-range transfers

## Register map (word registers, byte offsets)
| Offset | Name   | Description                                              |
|--------|--------|----------------------------------------------------------|
| 0x00   | SRC    | Source byte address (word aligned)                       |
| 0x04   | DST    | Destination byte address (word aligned)                  |
| 0x08   | LEN    | Number of 32-bit words to copy (1 or more)               |
| 0x0C   | CTRL   | bit0 START (ignored while BUSY), bit1 IRQ_EN             |
| 0x10   | STATUS | bit0 BUSY, bit1 DONE, bit2 ERR (read only)               |
| 0x14   | CLEAR  | Write 1 to clear: bit0 DONE, bit1 ERR                    |

## Project layout
- `rtl/dma_ctrl.sv`: DMA controller
- `rtl/dma_soc.sv`: DMA plus 4 KB RAM with a backdoor port for tests
- `tb/tb.sv`: SystemVerilog self-checking testbench
- `sw/`: C++ platform interface, C model, RTL driver, and shared tests

## Run
    pip install verilator        # also needs g++ and make
    make all                     # SV testbench, then C-model and RTL tests

Expected output: `RESULT: PASS` from the SystemVerilog testbench, and PASS on both platforms
(C-model and RTL) in the C++ run.

## Bug found by the tests
The first range check used 34 bits, so `addr + len*4` overflowed for `len = 0xFFFFFFFF` and the
transfer was wrongly accepted. I fixed it by widening the check to 35 bits.

## Limits
Not UVM. No real SoC, MMU, or IOMMU. The memory port is simple. Run on Verilator only, not on
FPGA or silicon.
