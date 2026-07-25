# Space-Craft-Hybrid v7.21 — Mermaid UML Diagram

```mermaid
classDiagram
    %% Core Mathematics Layer
    class Quaternion {
        +Double w
        +Double x
        +Double y
        +Double z
        +norm()
        +conjugate()
        +dot()
        +normalize()
    }

    class Vec3 {
        +Double x
        +Double y
        +Double z
        +norm()
        +scale()
        +add()
    }

    class UnitQuaternion {
        +Quaternion q
        +mkUnitQuaternion()
        +fromUnitQuaternion()
        +compose()
    }

    class InertiaTensor {
        +Double Ixx
        +Double Iyy
        +Double Izz
    }

    %% Lie Algebra
    class ExponentialMap {
        +exponentialMap()
    }

    class LogarithmicMap {
        +logarithmicMap()
    }

    %% Spacecraft State
    class SpacecraftState {
        +Vec3 position
        +Vec3 velocity
        +UnitQuaternion attitude
        +Vec3 angularVelocity
        +Double time
    }

    class SensorReading {
        +UnitQuaternion starTrackerAttitude
        +Vec3 imuAngularVelocity
        +simulateSensor()
    }

    %% Control System
    class ControlCommand {
        +Vec3 torque
        +Vec3 thrustVector
    }

    class ControlMetrics {
        +Double cmErrorMag
        +Double cmEnergyRatio
        +String cmRegime
        +Double cmKp
        +Double cmKd
        +Boolean cmBrakingActive
    }

    class HybridAttitudeControl {
        +hybridAttitudeControl()
    }

    %% Dynamics
    class AngularAcceleration {
        +angularAcceleration()
    }

    class GeometricIntegrator {
        +integrateGeometric()
    }

    %% Mission
    class MissionSimulation {
        +simulationStep()
    }

    class MissionRunner {
        +runMission()
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
