# Space-Craft-Hybrid v7.21 — Mermaid UML Diagram

```mermaid
classDiagram
    %% Core Mathematics Layer
    class Quaternion {
        Double w
        Double x
        Double y
        Double z
        norm() Double
        conjugate() Quaternion
        dot(Quaternion) Double
        normalize() Quaternion
    }

    class Vec3 {
        Double x
        Double y
        Double z
        norm() Double
        scale(Double) Vec3
        add(Vec3) Vec3
    }

    class UnitQuaternion {
        Quaternion q
        mkUnitQuaternion(Quaternion)* UnitQuaternion
        fromUnitQuaternion()* Quaternion
        compose(UnitQuaternion) UnitQuaternion
    }

    class InertiaTensor {
        Double Ixx = 100.0
        Double Iyy = 120.0
        Double Izz = 80.0
        Note: kg·m²
    }

    %% Lie Algebra
    class ExponentialMap {
        exponentialMap(Vec3)* UnitQuaternion
        Note: Tangent space → SO(3) rotation
    }

    class LogarithmicMap {
        logarithmicMap(UnitQuaternion)* Vec3
        Note: SO(3) rotation → tangent space
    }

    %% Spacecraft State
    class SpacecraftState {
        Vec3 position
        Vec3 velocity
        UnitQuaternion attitude
        Vec3 angularVelocity
        Double time
    }

    class SensorReading {
        UnitQuaternion starTrackerAttitude
        Vec3 imuAngularVelocity
        simulateSensor(SpacecraftState)* SensorReading
        Note: Adds jitter, drift, noise
    }

    %% Control System
    class ControlCommand {
        Vec3 torque
        Vec3 thrustVector
        Note: torque maxed at 45 N·m
    }

    class ControlMetrics {
        Double cmErrorMag
        Double cmEnergyRatio
        String cmRegime
        Double cmKp
        Double cmKd
        Boolean cmBrakingActive
    }

    class HybridAttitudeControl {
        hybridAttitudeControl(UnitQuaternion, UnitQuaternion, Vec3)*
        (ControlCommand, ControlMetrics)
        Note: 4-regime gain scheduling + energy-ratio braking
    }

    %% Dynamics
    class AngularAcceleration {
        angularAcceleration(InertiaTensor, Vec3, Vec3)* Vec3
        Note: Euler equations with gyroscopic coupling
    }

    class GeometricIntegrator {
        integrateGeometric(Double, InertiaTensor, ControlCommand, SpacecraftState)* SpacecraftState
        Note: Exponential map on SO(3)
    }

    %% Mission
    class MissionSimulation {
        simulationStep(UnitQuaternion, SpacecraftState)* 
        IO(SpacecraftState, ControlCommand, ControlMetrics)
        Note: 0.01s time step (100 Hz)
    }

    class MissionRunner {
        runMission(Int, UnitQuaternion, SpacecraftState)*
        IO List(Trajectory)
        Note: 2500 steps = 25 seconds
    }

    %% Relationships
    UnitQuaternion --> Quaternion
    UnitQuaternion --|> ExponentialMap
    UnitQuaternion --|> LogarithmicMap
    
    SpacecraftState --> UnitQuaternion
    SpacecraftState --> Vec3
    
    SensorReading --> UnitQuaternion
    SensorReading --> Vec3
    SensorReading --|> SpacecraftState
    
    HybridAttitudeControl --> UnitQuaternion
    HybridAttitudeControl --> Vec3
    HybridAttitudeControl --> ControlCommand
    HybridAttitudeControl --> ControlMetrics
    HybridAttitudeControl --|> LogarithmicMap
    
    AngularAcceleration --> InertiaTensor
    AngularAcceleration --> Vec3
    
    GeometricIntegrator --> ExponentialMap
    GeometricIntegrator --> AngularAcceleration
    
    MissionSimulation --> SensorReading
    MissionSimulation --> HybridAttitudeControl
    MissionSimulation --> GeometricIntegrator
    MissionSimulation --> SpacecraftState
    
    MissionRunner --> MissionSimulation
```

## Key Control Features

### Four Regimes (Gain Scheduling)
| Regime | Error Range | kp | kd | Purpose |
|--------|-------------|----|----|---------|
| Acquisition | > 1.5 rad (85°) | 8.0 | 12.0 | Fast slew |
| Tracking | 0.5–1.5 rad | 6.0 | 9.5 | Controlled approach |
| Settling | 0.1–0.5 rad | 4.5 | 9.0 | Smooth convergence |
| Fine-Point | < 0.1 rad (6°) | 8.0 | 24.0 | Precision hold |

### Energy-Ratio Braking
- **Kinetic energy ratio** = ω² / |error|
- If ratio > 0.08: apply 5× + 4×ratio multiplier to kd
- If ratio > 0.01: apply 3.5× + 2.5×ratio multiplier to kd
- **Goal**: Prevent overshoot and enforce monotonic convergence

### Quaternion Double-Cover Avoidance
- Checks dot product of desired · current
- If negative, negates target to take shortest path
- Prevents 360° unwinding
