# UML Diagrams for Spacecraft Projects

All diagrams are provided in **both PlantUML and Mermaid formats** so you can use them anywhere.

## Files Included

### 1. Space-Craft-Hybrid v7.21 (Attitude Control Simulator)

#### PlantUML
- **File**: `spacecraft_hybrid_plantuml.puml`
- **Usage**: Copy into [PlantUML Online Editor](http://www.plantuml.com/plantuml/uml) or use in your IDE (VS Code + PlantUML extension, IntelliJ, etc.)
- **Shows**:
  - Core quaternion math (Quaternion, Vec3, UnitQuaternion, InertiaTensor)
  - Lie algebra mappings (exponentialMap, logarithmicMap)
  - Spacecraft state & sensor simulation
  - Hybrid attitude control system with 4-regime gain scheduling
  - Dynamics integration (angular acceleration, geometric integrator)
  - Mission simulation loop

#### Mermaid
- **File**: `spacecraft_hybrid_mermaid.md`
- **Usage**: Paste into [Mermaid Live Editor](https://mermaid.live), GitHub markdown, or any Mermaid-compatible tool
- **Shows**: Same information, simpler syntax, GitHub-friendly
- **Bonus**: Includes control regimes table and energy-ratio braking explanation

---

### 2. Quaternion Memory Hardware (FPGA + C Driver)

#### PlantUML
- **File**: `quaternion_memory_hardware_plantuml.puml`
- **Usage**: Copy into [PlantUML Online Editor](http://www.plantuml.com/plantuml/uml) or IDE
- **Shows**:
  - FPGA hardware layer (state machine, BRAM storage, testbench)
  - Hardware abstraction (memory-mapped I/O)
  - Embedded C driver layer (SimulatedHardwareBank, API functions)
  - Component relationships and signal flow

#### Mermaid
- **File**: `quaternion_memory_hardware_mermaid.md`
- **Usage**: [Mermaid Live Editor](https://mermaid.live), GitHub markdown, docs
- **Shows**:
  - System architecture graph
  - C API function signatures
  - State machine diagram
  - Circular buffer behavior table
  - Data flow example walkthrough

---

## How to Use These Files

### Option 1: View Online (No Software Needed)
1. **PlantUML**: Go to [Plant UML Online](http://www.plantuml.com/plantuml/uml), paste the `.puml` file content
2. **Mermaid**: Go to [Mermaid Live](https://mermaid.live), paste the code block from the `.md` file

### Option 2: Use in Your IDE
- **VS Code**: Install "PlantUML" extension (jebbs.plantuml) + Graphviz, or use Mermaid preview
- **IntelliJ/PyCharm**: Built-in PlantUML support; right-click → Diagrams → Show Diagram
- **GitHub**: Paste Mermaid code directly into README.md (renders automatically)

### Option 3: Export as Image
**PlantUML**:
```bash
# Command line (requires Java + Graphviz)
plantuml spacecraft_hybrid_plantuml.puml -o ./images/
# Outputs: SVG, PNG, PDF
```

**Mermaid CLI**:
```bash
# Install: npm install -g mermaid-cli
mmdc -i quaternion_memory_hardware_mermaid.md -o diagrams.png
```

### Option 4: Embed in Documentation
**GitHub README.md**:
```markdown
## Architecture

```mermaid
%% Paste your Mermaid code here
%% GitHub will render it automatically
```
```

**Sphinx/Readthedocs**:
```rst
.. uml::
   :align: center

   @startuml
   %% Paste your PlantUML code here
   @enduml
```

---

## Diagram Overview

### Space-Craft-Hybrid v7.21

**Purpose**: Understand the flow from spacecraft state → sensor readings → attitude control → dynamics integration

**Key Takeaways**:
- Quaternions + Lie algebra = smooth, singularity-free rotations on SO(3)
- Four gain-scheduling regimes adapt control law to error magnitude
- Kinetic-to-potential energy ratio triggers extreme braking to prevent overshoot
- Geometric integrator preserves rotation manifold structure
- Mission runner orchestrates 100 Hz simulation loop (0.01s steps)

**Who cares**: Control systems engineers, simulation developers, spacecraft GNC teams

---

### Quaternion Memory Hardware

**Purpose**: Understand the FPGA↔C hardware-software boundary for quaternion snapshot recording

**Key Takeaways**:
- FPGA controller implements deterministic state machine + circular BRAM buffer
- C driver abstracts hardware as two simple functions: `q_bank_add()`, `q_bank_read()`
- Memory-mapped I/O (MMIO) registers bridge hardware and software
- Busy flag prevents write collisions; async reads don't block
- Automatic circular buffer wrap at 1024 entries (20 KB BRAM for 5 components × 32-bit)

**Who cares**: Embedded systems designers, FPGA developers, spacecraft flight software engineers

---

## Customization Tips

### PlantUML
- Change colors: `skinparam classBackgroundColor #e3f2fd`
- Adjust layout: Add `!direction right-to-left` for left-to-right flow
- Add notes: `note left of ClassName : Your text here`
- Add stereotypes: `<<interface>>`, `<<abstract>>`, `<<enum>>`

### Mermaid
- Adjust graph direction: `graph LR` (left-to-right) instead of `graph TB` (top-to-bottom)
- Use `subgraph` to group related components
- Link styling: `linkStyle 0 stroke:red`
- Class styling: `class A,B,C classA`

---

## Integration with Your Codebase

### Haskell (Space-Craft-Hybrid)
- Document the PlantUML class diagram in your README or Wiki
- Link to the Mermaid version for GitHub-friendly rendering
- Add type signatures from the diagram directly to Haddock comments

### Verilog + C (Quaternion Memory Hardware)
- Include the component diagram in your design documentation
- Use the state machine diagram for simulation test coverage justification
- Reference the C API signatures in the driver header file comments

---

## Questions?

- **PlantUML Docs**: https://plantuml.com/
- **Mermaid Docs**: https://mermaid.js.org/
- **UML Reference**: https://www.uml-diagrams.org/

Good luck with the spacecraft! 🚀
