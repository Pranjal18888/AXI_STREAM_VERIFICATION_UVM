# AXI_STREAM_VERIFICATION_UVM





# AXI4-Stream Asynchronous FIFO | Protocol-Based RTL Verification

A dual-clock **AXI4-Stream Asynchronous FIFO** designed in Verilog/SystemVerilog, with a complete **UVM-based verification environment**. The FIFO safely transfers streaming data between two unrelated clock domains using Gray-code pointers and 2-flop synchronizers.

## Features

- AXI4-Stream interface (`TDATA`, `TVALID`, `TREADY`, `TLAST`)
- Independent write clock (`s_aclk`) and read clock (`m_aclk`)
- Gray-code read/write pointers for safe CDC
- 2-flop synchronizers for pointer crossing
- Full / Empty flag generation
- `TLAST` (packet boundary) support
- Independent resets for both clock domains
- UVM testbench with sequences, driver, monitor, scoreboard, assertions and functional coverage

## Architecture

```
 Write Domain (s_aclk)                         Read Domain (m_aclk)
┌───────────────────────┐                    ┌────────────────────────┐
│ AXI-S Slave Interface │                    │ AXI-S Master Interface │
│  Write Pointer (bin)  │                    │  Read Pointer (bin)    │
│  Bin → Gray           │                    │  Bin → Gray            │
│  Full Logic           │                    │  Empty Logic           │
└──────────┬────────────┘                    └───────────┬────────────┘
           │            ┌─────────────────┐              │
           └──────────► │  Dual-Port Mem  │ ◄────────────┘
                        └─────────────────┘
   wptr_gray ──► [2-FF sync] ──► read domain
   rptr_gray ──► [2-FF sync] ──► write domain
```

### CDC Strategy

| Signal | Direction | Method |
|---|---|---|
| Write pointer (Gray) | Write → Read | 2-flop synchronizer, used for **Empty** |
| Read pointer (Gray) | Read → Write | 2-flop synchronizer, used for **Full** |

Only one bit changes per increment in Gray code, so a sampled pointer is never off by more than one location. This prevents metastability from producing a corrupted pointer value.

## Project Structure

```
├── rtl/
│   ├── async_fifo.v          # Top-level AXI4-Stream async FIFO
│   ├── sync_2ff.v            # 2-flop synchronizer
│   ├── wptr_full.v           # Write pointer + full logic
│   ├── rptr_empty.v          # Read pointer + empty logic
│   └── fifo_mem.v            # Dual-port memory
├── tb/
│   ├── axis_if.sv            # AXI4-Stream interface
│   ├── axis_seq_item.sv      # Sequence item
│   ├── axis_sequence.sv      # Sequences
│   ├── axis_driver.sv        # Driver
│   ├── axis_monitor.sv       # Monitor
│   ├── axis_scoreboard.sv    # Scoreboard
│   ├── axis_coverage.sv      # Functional coverage
│   ├── axis_assertions.sv    # SVA checks
│   ├── axis_agent.sv
│   ├── axis_env.sv
│   ├── axis_test.sv
│   └── tb_top.sv             # Top-level testbench
└── README.md
```

## Parameters

| Parameter | Description | Default |
|---|---|---|
| `DATA_WIDTH` | Width of `TDATA` | 8 / 32 |
| `ADDR_WIDTH` | FIFO depth = 2^ADDR_WIDTH | 4 |

## Verification Environment (UVM)

| Component | Role |
|---|---|
| **Sequence / Sequence Item** | Generates randomized and directed AXI-S packets |
| **Driver** | Drives data on the write side following TVALID/TREADY handshake |
| **Monitor** | Samples transactions on both write and read interfaces |
| **Scoreboard** | Compares written vs. read data (in-order, no loss/duplication) |
| **Assertions (SVA)** | Handshake and protocol rule checks |
| **Coverage** | Full/empty hits, back-pressure, TLAST, reset scenarios |

### What is Verified

- **Handshaking**: data held stable while `TVALID && !TREADY`
- **Data integrity**: every written beat is read in the same order
- **TLAST**: packet boundaries preserved end-to-end
- **CDC behavior**: different write/read clock ratios (fast→slow, slow→fast, equal)
- **Stalls / back-pressure**: FIFO full and empty conditions
- **Reset behavior**: reset in the middle of traffic

## Tools

- EDA Playground
- Verilog / SystemVerilog
- UVM

## How to Run

### On EDA Playground

1. Open [EDA Playground](https://www.edaplayground.com/).
2. Set **Testbench + Design** to `SystemVerilog/Verilog` and **UVM/OVM** to `UVM 1.2`.
3. Pick a simulator that supports UVM (e.g., Synopsys VCS / Aldec Riviera-PRO / Cadence Xcelium).
4. Paste RTL in `design.sv` and the testbench in `testbench.sv`.
5. Enable **Open EPWave after run** to view waveforms.
6. Click **Run**.

## Results

- All tests passing, scoreboard mismatches: **0**
- Functional coverage: **XX %**
- Assertions: **no failures**

## Key Concepts

`AXI4-Stream` · `Clock Domain Crossing (CDC)` · `Asynchronous FIFO` · `Gray Code` · `2-FF Synchronizer` · `UVM` · `SystemVerilog Assertions` · `Functional Coverage`

## Future Work

- Add almost-full / almost-empty flags
- Add parameterizable `TKEEP`, `TUSER`, `TID`
- Formal verification of CDC properties