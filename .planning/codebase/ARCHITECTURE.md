<!-- refreshed: 2026-06-22 -->
# Architecture

**Analysis Date:** 2026-06-22

## System Overview

```text
┌─────────────────────────────────────────────────────────────────────┐
│                     Robot (TimedRobot)                               │
│  `src/main/cpp/Robot.cpp` / `122-Swerve-Template/src/main/cpp/Robot.cpp` │
│  Entry point: main() → frc::StartRobot<Robot>()                     │
│  20ms periodic loop — delegates to CommandScheduler                  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│               CommandScheduler (WPILib singleton)                    │
│  Runs all registered Commands and calls Subsystem::Periodic()        │
└──────────┬────────────────────────────────────┬─────────────────────┘
           │ default commands / bound triggers   │ autonomous command
           ▼                                     ▼
┌──────────────────────────┐      ┌──────────────────────────────────┐
│  Subsystems              │      │  Commands / Autos                 │
│  `subsystems/SwerveDrive`│      │  `commands/` + PathPlanner autos  │
│  `subsystems/SwerveModule│      │  `commands/AutoWheelOffsets.cpp`  │
│  `subsystems/PoseEstimator`     │  `commands/Autos.cpp`             │
│  `subsystems/Elevator`   │      └──────────────────────────────────┘
│  `subsystems/Wrist`      │
│  `subsystems/Climber`    │
│  `subsystems/LEDController`
│  `subsystems/LED_Groups` │
│  `subsystems/Turret`     │
│  `subsystems/TurretIntake`
│  `subsystems/Turret_Shooter`
└──────────┬───────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Hardware Abstraction (via CTRE Phoenix6 / REVLib / WPILib HAL)      │
│  TalonFX motors, CANcoders, Pigeon2 IMU, NavX, PDP/PDH              │
└─────────────────────────────────────────────────────────────────────┘
           │                         ▲
           ▼                         │
┌──────────────────────────────────────────────────────────────────────┐
│  NetworkTables / SmartDashboard / DataLog                            │
│  Vision data (base_link, note topics), POI persistence, telemetry    │
└──────────────────────────────────────────────────────────────────────┘
```

## Repository Layout

This repository contains two parallel C++ FRC robot programs:

| Directory | Purpose |
|-----------|---------|
| `src/` | Minimal WPILib template project (skeleton with ExampleSubsystem) |
| `122-Swerve-Template/` | Full competition swerve robot (FRC Team 122) |

The `122-Swerve-Template/` is the primary, feature-complete codebase. The root `src/` is a WPILib new-project scaffold that has not yet been extended.

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| `Robot` | TimedRobot lifecycle, periodic scheduler calls, button binding, dashboard update | `122-Swerve-Template/src/main/cpp/Robot.cpp` |
| `SwerveDrive` | Chassis drive, odometry, pose estimation fusion, PathPlanner integration | `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp` |
| `SwerveModule` | Per-wheel TalonFX drive+steer control, CANcoder feedback | `122-Swerve-Template/src/main/include/subsystems/SwerveModule.hpp` |
| `PoseEstimator` | Vision-derived pose via NetworkTables (camera node publishes transforms) | `122-Swerve-Template/src/main/include/subsystems/PoseEstimator.h` |
| `PoseFilter` | Eigen-based sliding-window stability filter for raw vision poses | `122-Swerve-Template/src/main/include/utils/PoseFilter.h` |
| `POIGenerator` | Store and retrieve field Points-of-Interest via NetworkTables | `122-Swerve-Template/src/main/include/utils/POIGenerator.h` |
| `AutoWheelOffsets` | Command that calibrates swerve module steering encoder offsets | `122-Swerve-Template/src/main/cpp/commands/AutoWheelOffsets.cpp` |
| `Elevator` | Elevator height control (currently commented out in Robot) | `122-Swerve-Template/src/main/cpp/subsystems/Elevator.cpp` |
| `Wrist` | Wrist angle control (currently commented out in Robot) | `122-Swerve-Template/src/main/cpp/subsystems/Wrist.cpp` |
| `Climber` | Climber subsystem (currently commented out in Robot) | `122-Swerve-Template/src/main/cpp/subsystems/Climber.cpp` |
| `LEDController` / `LED_Groups` | LED animation control (currently commented out) | `122-Swerve-Template/src/main/cpp/subsystems/LEDController.cpp` |
| `Constants.hpp` | All hardware IDs, PID gains, kinematic parameters, drive constants | `122-Swerve-Template/src/main/include/Constants.hpp` |

## Pattern Overview

**Overall:** WPILib Command-Based Robot (Declarative)

**Key Characteristics:**
- Subsystems encapsulate hardware and expose methods; commands consume subsystems
- The `CommandScheduler` singleton owns the run loop — subsystems register themselves and are called via `Periodic()`
- `Robot` acts as the composition root: it owns all subsystem instances and binds controller inputs to commands in `BindCommands()` / `ConfigureBindings()`
- In the `122-Swerve-Template`, `Robot` consolidates what is conventionally split into `Robot` + `RobotContainer` — all subsystem ownership and button binding lives in a single `Robot` class
- The root `src/` project follows the standard two-class split: `Robot` (lifecycle) + `RobotContainer` (subsystem wiring)

## Layers

**Robot Lifecycle Layer:**
- Purpose: Implements WPILib `TimedRobot` hooks (Init/Periodic per mode)
- Location: `122-Swerve-Template/src/main/cpp/Robot.cpp`, `src/main/cpp/Robot.cpp`
- Contains: Mode transitions, scheduler invocation, dashboard updates
- Depends on: Subsystems, CommandScheduler, PathPlanner AutoBuilder
- Used by: WPILib runtime (`frc::StartRobot<Robot>()`)

**Subsystem Layer:**
- Purpose: Hardware abstraction + state management per robot mechanism
- Location: `122-Swerve-Template/src/main/cpp/subsystems/`, `122-Swerve-Template/src/main/include/subsystems/`
- Contains: Motor controllers, sensors, kinematics, periodic telemetry
- Depends on: CTRE Phoenix6, WPILib HAL, NetworkTables, Constants.hpp
- Used by: Robot (owns instances), Commands (receive pointers)

**Command Layer:**
- Purpose: Sequences of actions that require subsystems for a bounded time
- Location: `122-Swerve-Template/src/main/cpp/commands/`, `122-Swerve-Template/src/main/include/commands/`
- Contains: `AutoWheelOffsets` (encoder calibration), PathPlanner auto commands
- Depends on: Subsystems (held as pointers), PathPlanner library
- Used by: Robot::BindCommands(), Robot::AutonomousInit()

**Utility Layer:**
- Purpose: Stateless math helpers and non-subsystem support classes
- Location: `122-Swerve-Template/src/main/include/utils/`, `122-Swerve-Template/src/main/cpp/utils/`
- Contains: `POIGenerator`, `PoseFilter`, `MathUtilNK` (in Constants.hpp)
- Depends on: Eigen, WPILib geometry, NetworkTables
- Used by: SwerveDrive, Robot

**Configuration Layer:**
- Purpose: Compile-time constants for all electrical IDs, PID gains, kinematic values
- Location: `122-Swerve-Template/src/main/include/Constants.hpp`, `src/main/include/Constants.hpp`
- Contains: Namespaced `constexpr` values for `ElectricalConstants`, `DriveConstants`, `ModuleConstants`, `MathUtilNK`
- Depends on: CTRE headers, WPILib units
- Used by: All subsystems and commands

## Data Flow

### Teleop Drive Path

1. Driver moves joystick — `Robot::CreateRobot()` lambda captures `m_driverController.GetRawAxis()` (`122-Swerve-Template/src/main/cpp/Robot.cpp:194`)
2. `MathUtilNK::calculateAxis()` applies deadband (`Constants.hpp:175`)
3. `m_swerveDrive.Drive(frc::ChassisSpeeds::FromFieldRelativeSpeeds(...))` converts field-relative input to robot-relative speeds (`SwerveDrive.hpp:70`)
4. `SwerveDrive` calls `SwerveDriveKinematics::ToSwerveModuleStates()` and invokes `SwerveModule::SetDesiredState()` on each of 4 modules
5. Each `SwerveModule` commands `TalonFX` drive motor (velocity PID) and steer motor (position PID) via CTRE Phoenix6

### Autonomous Path

1. `Robot::RobotInit()` calls `pathplanner::AutoBuilder::buildAutoChooser()` — populates SmartDashboard chooser (`Robot.cpp:27`)
2. `Robot::AutonomousInit()` calls `autoChooser.GetSelected()` and schedules the selected PathPlanner auto (`Robot.cpp:77-83`)
3. PathPlanner commands call `SwerveDrive::Drive()` with `ChassisSpeeds` via the registered `AutoBuilder` callback
4. `SwerveDrive::UpdateOdometry()` / `UpdatePoseEstimate()` fuse wheel odometry with vision in `frc::SwerveDrivePoseEstimator<4>`

### Vision Fusion Path

1. External vision coprocessor publishes pose transforms to NetworkTables topics `base_link`, `base_link_1`, `base_link_2`
2. `PoseEstimator::Periodic()` reads `robot2Object` and `robotPose` subscribers (`PoseEstimator.h:38-39`)
3. `SwerveDrive` subscribes to `base_link1Subscribe` / `base_link2Subscribe` double-array topics
4. `PoseFilter::IsPoseValid()` validates pose stability using Eigen sliding-window comparison (`PoseFilter.h:20`)
5. Validated pose is fed into `frc::SwerveDrivePoseEstimator<4>::AddVisionMeasurement()` inside `SwerveDrive::UpdatePoseEstimate()`

**State Management:**
- Each subsystem holds its own hardware state as private members
- No global mutable state outside subsystem instances owned by `Robot`
- NetworkTables acts as the IPC bus between robot code and vision/dashboard systems

## Key Abstractions

**SubsystemBase:**
- Purpose: WPILib base class — registers subsystem with scheduler, provides `Periodic()` hook
- Examples: `SwerveDrive`, `SwerveModule`, `PoseEstimator`, all mechanism subsystems
- Pattern: Inherit `frc2::SubsystemBase`, override `Periodic()` and optionally `SimulationPeriodic()`

**CommandHelper / CommandPtr:**
- Purpose: Decorator-compatible command wrapper enabling `.AndThen()`, `.Until()` etc.
- Examples: `ExampleCommand` (`src/main/include/commands/ExampleCommand.hpp:19`), `AutoWheelOffsets`
- Pattern: Inherit `frc2::CommandHelper<frc2::Command, DerivedCommand>`; use `.ToPtr()` to get owning `CommandPtr`

**SwerveModule:**
- Purpose: Encapsulates one swerve corner — drive TalonFX, steer TalonFX, CANcoder
- Examples: `SwerveDrive` holds `std::array<SwerveModule, 4> modules` (`SwerveDrive.hpp:119`)
- Pattern: Constructed with CAN IDs and angle offset; exposes `SetDesiredState()` / `GetPosition()`

**Constants Namespaces:**
- Purpose: Centralize all magic numbers under typed namespaces
- Examples: `ElectricalConstants::kFrontLeftDriveMotorID`, `DriveConstants::kMaxTranslationalVelocity`, `ModuleConstants::kDriveP`
- Pattern: Header-only `const` / `constexpr` values in named namespaces inside `Constants.hpp`

## Entry Points

**Robot Program Entry:**
- Location: `main()` at bottom of `122-Swerve-Template/src/main/cpp/Robot.cpp:279`
- Triggers: Deployed binary executed by roboRIO/SystemCore on robot startup
- Responsibilities: Calls `frc::StartRobot<Robot>()` which starts the WPILib scheduler loop

**Simulation Entry:**
- Location: Same `main()` guarded by `#ifndef RUNNING_FRC_TESTS`
- Triggers: `./gradlew simulateNative`

**Test Entry:**
- Location: `src/test/cpp/main.cpp`, `122-Swerve-Template/src/test/cpp/main.cpp`
- Triggers: `./gradlew test` via GoogleTest framework

## Architectural Constraints

- **Threading:** Single-threaded event loop — all subsystem `Periodic()` calls and command execution happen synchronously at 20ms intervals on the main robot thread. `AddPeriodic()` can schedule callbacks at custom intervals (used for elevator/wrist at 5ms/10ms in commented code).
- **Global state:** `frc2::CommandScheduler::GetInstance()` is a WPILib singleton. `nt::NetworkTableInstance::GetDefault()` is a singleton. No other module-level global mutable state.
- **Circular imports:** None detected. `SwerveDrive` depends on `SwerveModule`; no reverse dependency.
- **Hardware IDs:** All CAN bus device IDs must match physical robot wiring. Defined in `ElectricalConstants` namespace in `Constants.hpp`. The Pigeon2 IMU uses CAN bus name `"NKCANivore"` (`SwerveDrive.hpp:116`).
- **Commented-out subsystems:** `Elevator`, `Wrist`, `Climber`, `LEDController` are instantiated and imported in `Robot.hpp` but commented out in `Robot.cpp`. Re-enabling requires uncommenting both declaration usage and `BindCommands()` entries.

## Anti-Patterns

### All robot wiring in Robot.cpp (122-Swerve-Template)

**What happens:** `Robot.cpp` owns subsystem instances, binds buttons, and handles all teleop input in a single class, replacing the standard `RobotContainer` split.
**Why it's wrong:** The file grows large and conflates lifecycle management with subsystem wiring. The root `src/` project correctly uses the two-class split.
**Do this instead:** Extract button bindings and subsystem declarations into a `RobotContainer` class and have `Robot` delegate to it, as demonstrated in `src/main/cpp/RobotContainer.cpp`.

### Vision subscriber topics hardcoded as string_view literals

**What happens:** Topic names like `"base_link"`, `"note"`, `"vision_stddev"` are scattered as `std::string_view` member literals in `SwerveDrive.hpp:142-145` and `PoseEstimator.h:35-36`.
**Why it's wrong:** Changing the topic name requires finding and updating multiple locations; no single source of truth.
**Do this instead:** Consolidate NetworkTable topic name constants into `Constants.hpp` under a `VisionConstants` or `NetworkTableConstants` namespace.

## Error Handling

**Strategy:** WPILib convention — hardware errors are reported via `frc::DriverStation` error/warning API and logged to DataLog. No exceptions thrown in robot code.

**Patterns:**
- `std::optional<frc2::CommandPtr>` guards the autonomous command before scheduling (`Robot.hpp:71`) — checked with `if (m_autonomousCommand)` before cancel/schedule
- `PoseFilter::IsPoseValid()` returns `bool` — caller discards invalid vision frames rather than asserting
- DataLog entries for PDP voltage/current/power/energy are written every robot periodic cycle for post-match analysis

## Cross-Cutting Concerns

**Logging:** `frc::DataLogManager` (WPILib DataLog) initialized in `Robot::RobotInit()`. PDP telemetry logged via `wpi::log::DoubleLogEntry`. SmartDashboard used for real-time dashboard values.

**Validation:** `PoseFilter` provides sliding-window pose validation for vision inputs. Joystick axis deadband applied via `MathUtilNK::calculateAxis()` before any drive calculations.

**Authentication:** Not applicable — FRC robots communicate on a private field network. No authentication layer.

---

*Architecture analysis: 2026-06-22*
