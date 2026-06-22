<!-- refreshed: 2026-06-22 -->
# Architecture

**Analysis Date:** 2026-06-22

## System Overview

```text
┌─────────────────────────────────────────────────────────────────┐
│                      main() Entry Point                         │
│           `src/main/cpp/Robot.cpp` (wpi::StartRobot<Robot>)     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Robot : wpi::TimedRobot                       │
│                   `src/main/cpp/Robot.cpp`                      │
│                                                                 │
│  RobotPeriodic() → CommandScheduler::GetInstance().Run()        │
│  AutonomousInit() → container.GetAutonomousCommand()            │
│  TeleopInit()    → autonomousCommand->Cancel()                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ owns (member)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     RobotContainer                              │
│               `src/main/cpp/RobotContainer.cpp`                 │
│                                                                 │
│  - Declares all Subsystems as members                           │
│  - Declares all input devices (CommandGamepad)                  │
│  - ConfigureBindings() maps Triggers → Commands                 │
│  - GetAutonomousCommand() returns selected auto sequence        │
└──────────┬──────────────────────────────┬───────────────────────┘
           │ owns (member)                │ creates
           ▼                              ▼
┌──────────────────────┐      ┌───────────────────────────────────┐
│  ExampleSubsystem    │      │  Commands / Autos                 │
│  `src/main/cpp/      │      │  `src/main/cpp/commands/`         │
│   subsystems/        │      │                                   │
│   ExampleSubsystem   │      │  ExampleCommand — requires        │
│   .cpp`              │      │    ExampleSubsystem               │
│                      │      │  autos::ExampleAuto — Sequence    │
│  Extends             │      │    of ExampleMethodCommand +      │
│  SubsystemBase       │      │    ExampleCommand                 │
│  Periodic() called   │◄─────│  Commands receive subsystem ptr   │
│  by scheduler        │      └───────────────────────────────────┘
└──────────────────────┘
           ▲
           │ drives via Trigger bindings + direct scheduling
┌──────────────────────┐
│  CommandGamepad      │
│  (driverController)  │
│  Port 0              │
└──────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| `main()` | Bootstraps the WPILib runtime with `wpi::StartRobot<Robot>()` | `src/main/cpp/Robot.cpp` |
| `Robot` | Implements TimedRobot lifecycle hooks; drives CommandScheduler each 20 ms loop | `src/main/cpp/Robot.cpp` / `src/main/include/Robot.hpp` |
| `RobotContainer` | Single source of truth for all subsystems, controllers, and trigger-to-command bindings | `src/main/cpp/RobotContainer.cpp` / `src/main/include/RobotContainer.hpp` |
| `ExampleSubsystem` | Owns hardware devices; exposes state queries and command factory methods; Periodic() runs each loop | `src/main/cpp/subsystems/ExampleSubsystem.cpp` / `src/main/include/subsystems/ExampleSubsystem.hpp` |
| `ExampleCommand` | Encapsulates a single robot action; declares subsystem requirements via `AddRequirements()` | `src/main/cpp/commands/ExampleCommand.cpp` / `src/main/include/commands/ExampleCommand.hpp` |
| `autos::ExampleAuto` | Composes a sequential autonomous routine from subsystem method commands and named commands | `src/main/cpp/commands/Autos.cpp` / `src/main/include/commands/Autos.hpp` |
| `Constants` | Header-only namespace for robot-wide numeric constants (`OperatorConstants::kDriverControllerPort`) | `src/main/include/Constants.hpp` |

## Pattern Overview

**Overall:** WPILib Command-Based (Commands V2)

**Key Characteristics:**
- Declarative paradigm: logic lives in Commands and Subsystems, not in Robot periodic methods
- CommandScheduler is the central runtime loop — called once per 20 ms in `RobotPeriodic()`
- Subsystems enforce mutual exclusion: a Command declares which Subsystems it `AddRequirements()`, and the scheduler enforces that only one command runs on a subsystem at a time
- Triggers (button presses, sensor conditions) bind input events to Commands declaratively in `RobotContainer::ConfigureBindings()`
- Autonomous routines are `CommandPtr` values composed with combinators (`wpi::cmd::Sequence`, etc.)

## Layers

**Robot (Framework Bridge):**
- Purpose: Implements WPILib's `TimedRobot` interface; translates mode lifecycle (Autonomous, Teleop, Disabled) into scheduler operations
- Location: `src/main/cpp/Robot.cpp`, `src/main/include/Robot.hpp`
- Contains: Lifecycle hooks, `autonomousCommand` optional, `RobotContainer` member
- Depends on: `RobotContainer`, `wpi::cmd::CommandScheduler`
- Used by: WPILib runtime via `wpi::StartRobot<Robot>()`

**RobotContainer (Wiring Layer):**
- Purpose: Owns all subsystem instances and input devices; wires triggers to commands
- Location: `src/main/cpp/RobotContainer.cpp`, `src/main/include/RobotContainer.hpp`
- Contains: Subsystem member declarations, `CommandGamepad` declaration, `ConfigureBindings()`, `GetAutonomousCommand()`
- Depends on: All subsystem headers, command headers, Constants
- Used by: `Robot` (holds as private member)

**Subsystems (Hardware Abstraction):**
- Purpose: Encapsulate all hardware device objects (motors, sensors, encoders); expose state and command factory methods; `Periodic()` runs every scheduler tick
- Location: `src/main/cpp/subsystems/`, `src/main/include/subsystems/`
- Contains: Classes extending `wpi::cmd::SubsystemBase`
- Depends on: WPILib hardware vendor libraries
- Used by: `RobotContainer` (ownership), Commands (via raw pointer requirements)

**Commands (Action Layer):**
- Purpose: Encode discrete robot behaviors; require subsystems; compose into sequences and groups
- Location: `src/main/cpp/commands/`, `src/main/include/commands/`
- Contains: Classes extending `wpi::cmd::CommandHelper<wpi::cmd::Command, DerivedType>`, standalone auto factory functions
- Depends on: Subsystem headers
- Used by: `RobotContainer` (bound to triggers or returned as auto command)

**Constants (Configuration):**
- Purpose: Centralize numeric and boolean constants in named namespaces
- Location: `src/main/include/Constants.hpp`
- Contains: `namespace OperatorConstants`, `namespace DriveConstants`, etc. (as project grows)
- Depends on: Nothing
- Used by: `RobotContainer`, Subsystems, Commands

## Data Flow

### Robot Startup

1. `main()` calls `wpi::StartRobot<Robot>()` — `src/main/cpp/Robot.cpp:79`
2. `Robot::Robot()` constructs `RobotContainer container` — `src/main/include/Robot.hpp:32`
3. `RobotContainer::RobotContainer()` constructs all subsystems and calls `ConfigureBindings()` — `src/main/cpp/RobotContainer.cpp:11`
4. `ConfigureBindings()` registers `Trigger → Command` mappings with the CommandScheduler — `src/main/cpp/RobotContainer.cpp:18`

### Periodic Loop (20 ms)

1. WPILib runtime calls `Robot::RobotPeriodic()` — `src/main/cpp/Robot.cpp:19`
2. `CommandScheduler::GetInstance().Run()` executes — `src/main/cpp/Robot.cpp:20`
3. Scheduler polls all registered `Trigger` conditions
4. Scheduler calls `Periodic()` on every registered `SubsystemBase`
5. Scheduler runs `Execute()` on each active Command; calls `End()` when `IsFinished()` returns true

### Autonomous Flow

1. `Robot::AutonomousInit()` calls `container.GetAutonomousCommand()` — `src/main/cpp/Robot.cpp:37`
2. `RobotContainer::GetAutonomousCommand()` returns `autos::ExampleAuto(&subsystem)` — `src/main/cpp/RobotContainer.cpp:31`
3. `autos::ExampleAuto` returns `wpi::cmd::Sequence(...)` composed from subsystem factory method and named command — `src/main/cpp/commands/Autos.cpp:10`
4. `CommandScheduler::Schedule()` queues the command — `src/main/cpp/Robot.cpp:40`
5. On `TeleopInit()`, `autonomousCommand->Cancel()` is called — `src/main/cpp/Robot.cpp:53`

### Teleop Trigger Flow

1. `Trigger([this] { return subsystem.ExampleCondition(); }).OnTrue(...)` — condition polled each scheduler tick — `src/main/cpp/RobotContainer.cpp:22`
2. `driverController.EastFace().WhileTrue(subsystem.ExampleMethodCommand())` — gamepad button held schedules inline command — `src/main/cpp/RobotContainer.cpp:28`

**State Management:**
- No global mutable state. Subsystem state is encapsulated within each `SubsystemBase` subclass instance owned by `RobotContainer`.
- `autonomousCommand` held as `std::optional<wpi::cmd::CommandPtr>` in `Robot` to safely handle the case where no auto is selected.

## Key Abstractions

**SubsystemBase:**
- Purpose: Base class for hardware groupings; auto-registers with CommandScheduler for `Periodic()` calls
- Examples: `src/main/include/subsystems/ExampleSubsystem.hpp`
- Pattern: Extend `wpi::cmd::SubsystemBase`, declare hardware members private, expose state via bool methods and commands via `CommandPtr` factory methods

**CommandHelper / CommandPtr:**
- Purpose: `CommandHelper<Base, Derived>` provides CRTP for command decorator methods (`.ToPtr()`, `.WithTimeout()`, etc.); `CommandPtr` is a type-erased owning handle
- Examples: `src/main/include/commands/ExampleCommand.hpp`
- Pattern: Extend `wpi::cmd::CommandHelper<wpi::cmd::Command, DerivedCommand>`, call `AddRequirements()` in constructor

**Trigger:**
- Purpose: Wraps any `bool`-returning callable; attaches Commands via `.OnTrue()`, `.WhileTrue()`, `.OnFalse()`
- Examples: `src/main/cpp/RobotContainer.cpp:22`
- Pattern: Declare inline in `ConfigureBindings()` using lambdas for sensor conditions, or obtain from `CommandGamepad` button accessors

## Entry Points

**Program Entry:**
- Location: `src/main/cpp/Robot.cpp:78`
- Triggers: Compiled binary executed by roboRIO/SystemCore OS at robot boot
- Responsibilities: Hands control to WPILib runtime

**Robot Lifecycle Hooks:**
- Location: `src/main/cpp/Robot.cpp`
- Triggers: WPILib runtime calls each hook when the FMS/DS signals a mode change
- Responsibilities: `AutonomousInit` schedules auto command; `TeleopInit` cancels it; `RobotPeriodic` ticks the scheduler

## Architectural Constraints

- **Threading:** Single-threaded event loop. All user code runs on the main robot thread via the CommandScheduler. Do not use raw threads in Commands or Subsystems without explicit synchronization.
- **Global state:** No module-level singletons in user code. `CommandScheduler` is a WPILib-managed singleton accessed via `GetInstance()`.
- **Circular imports:** None detected. Dependency direction is strict: Commands depend on Subsystems; RobotContainer depends on both; Robot depends only on RobotContainer.
- **Subsystem ownership:** Subsystems are owned (by value) inside `RobotContainer`. Commands receive raw pointers (`ExampleSubsystem*`) — the subsystem lifetime always exceeds command lifetime since both live for the robot session.
- **Command ownership:** `CommandPtr` is a move-only owning handle. Once `.ToPtr()` is called and the command is scheduled or stored, the original object is consumed.

## Anti-Patterns

### Logic in Robot periodic methods

**What happens:** Placing sensor reads, motor writes, or state machine logic directly in `Robot::TeleopPeriodic()` or `Robot::AutonomousPeriodic()`
**Why it's wrong:** Bypasses the scheduler's subsystem exclusion system, making concurrent command conflicts possible and code harder to test
**Do this instead:** Put all behavior in `Command::Execute()` or `SubsystemBase::Periodic()` and bind via Triggers in `RobotContainer::ConfigureBindings()`

### Extending Command directly instead of CommandHelper

**What happens:** `class MyCommand : public wpi::cmd::Command { ... }`
**Why it's wrong:** Decorator methods (`.WithTimeout()`, `.AndThen()`, `.ToPtr()`) will not work correctly without the CRTP provided by `CommandHelper`
**Do this instead:** `class MyCommand : public wpi::cmd::CommandHelper<wpi::cmd::Command, MyCommand>` — see `src/main/include/commands/ExampleCommand.hpp:18`

## Error Handling

**Strategy:** WPILib provides no exception-based error handling at runtime. Hardware failures surface via HAL error codes and `DataLog`/`SmartDashboard` reporting.

**Patterns:**
- `std::optional<wpi::cmd::CommandPtr>` used for `autonomousCommand` to safely handle null auto — `src/main/include/Robot.hpp:31`
- Subsystem constructors should validate hardware device initialization (check CAN IDs, encoder presence) and log errors to the Driver Station

## Cross-Cutting Concerns

**Logging:** WPILib `DataLog` / SmartDashboard / Shuffleboard APIs (not yet wired in this scaffold)
**Validation:** Constants defined in `src/main/include/Constants.hpp` namespaces; no runtime validation framework
**Authentication:** Not applicable (FRC robot program — no network auth layer)

---

*Architecture analysis: 2026-06-22*
