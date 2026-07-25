# Quaternion Memory Hardware — Mermaid Diagram

## System Architecture

```mermaid
graph TB
    subgraph FPGA["FPGA Hardware Layer (Verilog)"]
        Controller["QuaternionMemoryBank_Controller<br/>━━━━━━━━━━━━━━━━━<br/>State Machine: IDLE → SAVE → OUTPUT<br/>Write addr auto-increment (wrap at 1023)<br/>Async reads, sync writes"]
        
        BRAM["Block RAM Storage<br/>━━━━━━━━━━━━━━━━━<br/>mem_scalar: 1024×32b (float w)<br/>mem_x: 1024×32b (float x)<br/>mem_y: 1024×32b (float y)<br/>mem_z: 1024×32b (float z)<br/>mem_angle: 1024×32b (float θ)"]
        
        TB["Testbench (tb_QuaternionMemoryBank.v)<br/>━━━━━━━━━━━━━━━━━<br/>Instantiates controller<br/>Test scenarios: save → verify → read<br/>Generates VCD waveform"]
        
        Controller -->|read/write| BRAM
        TB -->|stimulates| Controller
    end
    
    subgraph HAL["Hardware Abstraction"]
        MMU["Memory-Mapped I/O Registers<br/>━━━━━━━━━━━━━━━━━<br/>CTRL_REG: pulse save bit<br/>STATUS_REG: busy flags<br/>SNAPSHOT_SEL: addr pointer<br/>Q_SCALAR/X/Y/Z/ANGLE: write path<br/>R_SCALAR/X/Y/Z/R_ANGLE: read path<br/>SNAPSHOT_COUNT[15:0]"]
    end
    
    subgraph C["Embedded C Driver Layer"]
        HwBank["SimulatedHardwareBank struct<br/>━━━━━━━━━━━━━━━━━<br/>Mirrors hardware registers<br/>Models MMIO in software<br/>write_addr counter<br/>Simulates BRAM for testing"]
        
        Add["q_bank_add() API<br/>━━━━━━━━━━━━━━━━━<br/>bool q_bank_add(<br/>  float s, x, y, z, angle)<br/><br/>1. Poll busy_flag<br/>2. Write registers<br/>3. Pulse CTRL_REG<br/>4. Increment snapshot_count<br/>5. Auto-increment write_addr<br/>6. Return success"]
        
        Read["q_bank_read() API<br/>━━━━━━━━━━━━━━━━━<br/>void q_bank_read(<br/>  uint32_t index,<br/>  float* s, x, y, z, angle)<br/><br/>1. Set READ_ADDR = index<br/>2. Read r_* registers<br/>3. Bit-cast uint32→float<br/>4. Dereference pointers"]
        
        Driver["main() Test Driver<br/>━━━━━━━━━━━━━━━━━<br/>Initialize hardware bank<br/>Call q_bank_add() ×N<br/>Call q_bank_read() verification<br/>Print snapshot contents"]
        
        HwBank -->|wraps| MMU
        Add -->|uses| HwBank
        Read -->|uses| HwBank
        Driver -->|calls| Add
        Driver -->|calls| Read
    end
    
    BRAM -->|synthesizes| MMU
    Controller -->|implements| MMU
    
    style FPGA fill:#e1bee7
    style C fill:#c8e6c9
    style HAL fill:#fff9c4
```

## API Function Signatures (C)

```c
// Write a quaternion snapshot to FPGA memory
bool q_bank_add(float s, float x, float y, float z, float angle);
// Returns: true if write succeeded, false if full/busy
// Side effects: increments snapshot_count, auto-wraps write_addr at 1023

// Read a quaternion snapshot by index
void q_bank_read(uint32_t index, float *s, float *x, float *y, float *z, float *angle);
// Inputs: index in [0, 1023]
// Outputs: dereferenced floats from BRAM at that address
// Note: bit-casts uint32 BRAM values to IEEE 754 floats
```

## Hardware State Machine (Controller)

```mermaid
stateDiagram-v2
    [*] --> IDLE
    
    IDLE --> IDLE: ctrl_save=0
    IDLE --> SAVE: ctrl_save=1 && !busy_flag
    
    SAVE --> OUTPUT: async_read_propagation
    
    OUTPUT --> IDLE: default (all paths)
    
    note right of SAVE
        busy_flag ← 1
        mem_*[write_addr] ← input
        write_addr ← write_addr + 1
        snapshot_count ← snapshot_count + 1
        busy_flag ← 0
    end note
    
    note right of OUTPUT
        r_scalar ← mem_scalar[read_addr]
        r_x ← mem_x[read_addr]
        r_y ← mem_y[read_addr]
        r_z ← mem_z[read_addr]
        r_angle ← mem_angle[read_addr]
    end note
```

## Circular Buffer Behavior

| Operation | Behavior |
|-----------|----------|
| **Write (q_bank_add)** | Stores (s,x,y,z,θ) at write_addr; increments; wraps at 1024 |
| **Read (q_bank_read)** | Async lookup at read_addr; bit-casts FP; returns via pointers |
| **Capacity** | 1024 entries × 5 quaternion components × 32-bit each = 20 KB BRAM |
| **Overflow** | write_addr wraps silently; oldest entries overwritten |
| **Concurrency** | Busy flag prevents simultaneous write; reads are always safe |

## Data Flow Example

```
C Code (spacecraft.c):
┌─────────────────────────────┐
│ float q_s = 0.7071, ...    │
│ q_bank_add(q_s, ...);      │ ──┐
└─────────────────────────────┘   │
                                   ├─→ SimulatedHardwareBank
                                   │   (mirrors registers)
                                   │
                                   ├─→ Q_SCALAR ← uint32_bits(0.7071)
                                   │   Q_X ← uint32_bits(x)
                                   │   ...
                                   │
                                   ├─→ CTRL_REG ← 0x01 (pulse save)
                                   │
                                   └─→ QuaternionMemoryBank_Controller
                                       (hardware or simulation)
                                       
                                       mem_scalar[write_addr] ← 0x3f34a3d7
                                       mem_x[write_addr] ← ...
                                       mem_y[...] ← ...
                                       mem_z[...] ← ...
                                       mem_angle[...] ← ...
                                       
                                       write_addr ← (write_addr + 1) % 1024
                                       snapshot_count ← snapshot_count + 1

Later, retrieve:
┌──────────────────────────────┐
│ float s_read, x_read, ...;  │
│ q_bank_read(0, &s_read, ...);│ ──┐
└──────────────────────────────┘   │
                                    ├─→ READ_ADDR ← 0
                                    │
                                    └─→ R_SCALAR = mem_scalar[0]
                                        s_read ← float_bits(R_SCALAR)
                                        x_read ← float_bits(R_X)
                                        ...
```

## Key Design Decisions

1. **Dual BRAM blocks** (separate s, x, y, z, angle): allows parallel reads across all components
2. **Circular buffer**: no external management needed; write_addr wraps automatically
3. **Async reads**: no pipeline stall while writing
4. **Busy flag** (single bit): lightweight synchronization without spinlocks
5. **32-bit IEEE 754 floats**: hardware-native FP format, zero conversion cost
6. **C driver abstraction**: hides register complexity; one API call = one snapshot
