# Codebase Structure

**Analysis Date:** 2026-06-22

> **Scope note:** This document covers only the root project. The `122-Swerve-Template/` directory is a git submodule from another team and is excluded entirely.

## Directory Layout

```
SwerveBaseTest/
├── src/                        # All robot source code
│   ├── main/
│   │   ├── cpp/                # C++ implementation files (.cpp)
│   │   │   ├── Robot.cpp       # TimedRobot lifecycle + main()
│   │   │   ├── RobotContainer.cpp  # Subsystem/command wiring
│   │   │   ├── commands/
│   │   │   │   ├── Autos.cpp       # Autonomous routine factory functions
│   │   │   │   └── ExampleCommand.cpp
│   │   │   └── subsystems/
│   │   │       └── ExampleSubsystem.cpp
│   │   ├── include/            # C++ header files (.hpp)
│   │   │   ├── Robot.hpp
│   │   │   ├── RobotContainer.hpp
│   │   │   ├── Constants.hpp   # Robot-wide constants (header-only)
│   │   │   ├── commands/
│   │   │   │   ├── Autos.hpp
│   │   │   │   └── ExampleCommand.hpp
│   │   │   └── subsystems/
│   │   │       └── ExampleSubsystem.hpp
│   │   └── deploy/             # Files deployed to robot filesystem
│   │       └── example.txt
│   └── test/
│       └── cpp/
│           └── main.cpp        # GoogleTest entry point for unit tests
├── vendordeps/                 # WPILib vendor dependency JSON declarations
│   └── CommandsV2.json
├── .wpilib/                    # WPILib IDE preferences (team number, language)
│   └── wpilib_preferences.json
├── build.gradle                # GradleRIO build configuration
├── settings.gradle             # Gradle project name
├── gradlew / gradlew.bat       # Gradle wrapper scripts
├── build/                      # Generated build artifacts (not committed)
└── 122-Swerve-Template/        # Git submodule — EXCLUDED from this project
```

## Directory Purposes

**`src/main/cpp/`:**
- Purpose: All C++ implementation files for the robot program
- Contains: `.cpp` files mirroring the header structure in `src/main/include/`
- Key files: `Robot.cpp` (entry point + lifecycle), `RobotContainer.cpp` (wiring)

**`src/main/include/`:**
- Purpose: All C++ header files; this directory is the exported include root
- Contains: `.hpp` headers organized into `commands/` and `subsystems/` subdirectories
- Key files: `Constants.hpp` (header-only constants), `Robot.hpp`, `RobotContainer.hpp`

**`src/main/include/subsystems/`** and **`src/main/cpp/subsystems/`:**
- Purpose: One subsystem per hardware group (drivetrain, shooter, intake, etc.)
- Contains: Classes extending `wpi::cmd::SubsystemBase`

**`src/main/include/commands/`** and **`src/main/cpp/commands/`:**
- Purpose: Discrete robot actions and autonomous routine factories
- Contains: Command classes extending `wpi::cmd::CommandHelper`, free functions in `autos::` namespace

**`src/main/deploy/`:**
- Purpose: Static files copied to the robot's filesystem at deploy time (path mappings, config JSONs, etc.)
- Deployed to: `/home/systemcore/deploy/` on the robot

**`src/test/cpp/`:**
- Purpose: GoogleTest unit tests for robot code; uses HAL simulation
- Key files: `main.cpp` (test runner bootstrap calling `HAL_Initialize`)

**`vendordeps/`:**
- Purpose: JSON descriptor files declaring third-party WPILib vendor libraries
- Key files: `CommandsV2.json` (Commands V2 library; conflicts with V3)

**`.wpilib/`:**
- Purpose: WPILib VS Code extension and GradleRIO preferences
- Key files: `wpilib_preferences.json` — stores team number (122), language (cpp), project year (2027_alpha5)

## Key File Locations

**Entry Points:**
- `src/main/cpp/Robot.cpp:78`: `main()` — calls `wpi::StartRobot<Robot>()`
- `src/main/cpp/Robot.cpp:19`: `Robot::RobotPeriodic()` — ticks the CommandScheduler

**Configuration:**
- `src/main/include/Constants.hpp`: All numeric/boolean robot constants, organized by namespace
- `build.gradle`: GradleRIO build targets, simulation config, desktop support flag
- `.wpilib/wpilib_preferences.json`: Team number and WPILib year
- `vendordeps/CommandsV2.json`: Commands V2 vendor dependency declaration

**Core Logic:**
- `src/main/cpp/RobotContainer.cpp`: All trigger-to-command bindings and auto selection
- `src/main/cpp/subsystems/ExampleSubsystem.cpp`: Hardware encapsulation template
- `src/main/cpp/commands/Autos.cpp`: Autonomous routine composition

**Testing:**
- `src/test/cpp/main.cpp`: GoogleTest runner entry point

## Naming Conventions

**Files:**
- Headers: `PascalCase.hpp` (e.g., `ExampleSubsystem.hpp`, `RobotContainer.hpp`)
- Implementations: `PascalCase.cpp` matching their header name
- Constants file: singular `Constants.hpp` — not per-subsystem files

**Directories:**
- Lowercase plural: `subsystems/`, `commands/`
- Mirror structure between `src/main/cpp/` and `src/main/include/` exactly

**Classes:**
- `PascalCase` for all classes: `ExampleSubsystem`, `ExampleCommand`, `RobotContainer`
- Command classes: named for the action they perform (verb + noun or verb phrase)
- Subsystem classes: named for the hardware group (noun)

**Namespaces:**
- Constants grouped in `PascalCase` namespaces: `namespace OperatorConstants`, `namespace DriveConstants`
- Constants use `k` prefix: `kDriverControllerPort`
- Autonomous factory functions live in `namespace autos`

**Methods:**
- WPILib lifecycle overrides: `PascalCase` matching WPILib interface (`Periodic`, `AutonomousInit`, `TeleopPeriodic`)
- Public subsystem APIs: `PascalCase` (e.g., `ExampleCondition()`, `ExampleMethodCommand()`)
- Private wiring: `ConfigureBindings()`

## Where to Add New Code

**New Subsystem (e.g., Drivetrain):**
- Header: `src/main/include/subsystems/Drivetrain.hpp` — extend `wpi::cmd::SubsystemBase`
- Implementation: `src/main/cpp/subsystems/Drivetrain.cpp`
- Register: Add `Drivetrain drivetrain;` as a private member in `src/main/include/RobotContainer.hpp`
- Constants: Add `namespace DriveConstants { ... }` in `src/main/include/Constants.hpp`

**New Command (e.g., DriveWithJoystick):**
- Header: `src/main/include/commands/DriveWithJoystick.hpp` — extend `CommandHelper<Command, DriveWithJoystick>`
- Implementation: `src/main/cpp/commands/DriveWithJoystick.cpp` — call `AddRequirements()` in constructor
- Bind: Wire to a Trigger in `RobotContainer::ConfigureBindings()` in `src/main/cpp/RobotContainer.cpp`

**New Autonomous Routine:**
- Add a new factory function `autos::MyAuto(...)` declaration in `src/main/include/commands/Autos.hpp`
- Implement in `src/main/cpp/commands/Autos.cpp` using `wpi::cmd::Sequence`, `Parallel`, etc.
- Return from `RobotContainer::GetAutonomousCommand()` (or via a `SendableChooser`)

**Robot-Wide Constant:**
- Add to `src/main/include/Constants.hpp` inside the appropriate namespace (create a new namespace if needed)

**Deploy Asset (path map, calibration file):**
- Place in `src/main/deploy/` — GradleRIO copies it to `/home/systemcore/deploy/` on deploy

**Vendor Library:**
- Add JSON descriptor to `vendordeps/` (use WPILib VS Code "Manage Vendor Libraries" tool)

**Unit Test:**
- Add `.cpp` files under `src/test/cpp/` — GoogleTest with HAL simulation is already configured

## Special Directories

**`build/`:**
- Purpose: Compiled object files and linked executables
- Generated: Yes
- Committed: No (add to `.gitignore`)

**`.gradle/`:**
- Purpose: Gradle daemon cache and dependency resolution artifacts
- Generated: Yes
- Committed: No

**`122-Swerve-Template/`:**
- Purpose: External git submodule — swerve drive template from team 122
- Generated: No (manually added submodule)
- Committed: Tracked as submodule reference only; contents not part of this project

---

*Structure analysis: 2026-06-22*
