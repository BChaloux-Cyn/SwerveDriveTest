# Codebase Structure

**Analysis Date:** 2026-06-22

## Directory Layout

```
SwerveBaseTest/                        # Repo root
├── src/                               # WPILib skeleton project (not yet extended)
│   ├── main/
│   │   ├── cpp/                       # C++ implementation files
│   │   │   ├── Robot.cpp              # Minimal TimedRobot lifecycle
│   │   │   ├── RobotContainer.cpp     # Subsystem wiring + button bindings
│   │   │   ├── commands/
│   │   │   │   ├── Autos.cpp          # Autonomous command factory
│   │   │   │   └── ExampleCommand.cpp # Example command implementation
│   │   │   └── subsystems/
│   │   │       └── ExampleSubsystem.cpp
│   │   ├── include/                   # C++ header files (mirrors cpp/ layout)
│   │   │   ├── Robot.hpp
│   │   │   ├── RobotContainer.hpp
│   │   │   ├── Constants.hpp          # Robot-wide constants
│   │   │   ├── commands/
│   │   │   │   ├── Autos.hpp
│   │   │   │   └── ExampleCommand.hpp
│   │   │   └── subsystems/
│   │   │       └── ExampleSubsystem.hpp
│   │   └── deploy/                    # Files deployed to robot (pathplanner paths, etc.)
│   └── test/
│       └── cpp/
│           └── main.cpp               # GoogleTest entry point
├── 122-Swerve-Template/               # Full competition robot (primary codebase)
│   ├── src/
│   │   ├── main/
│   │   │   ├── cpp/
│   │   │   │   ├── Robot.cpp          # Lifecycle + subsystem wiring + button bindings
│   │   │   │   ├── commands/
│   │   │   │   │   └── AutoWheelOffsets.cpp
│   │   │   │   ├── subsystems/
│   │   │   │   │   ├── SwerveDrive.cpp
│   │   │   │   │   ├── SwerveModule.cpp
│   │   │   │   │   ├── PoseEstimator.cpp
│   │   │   │   │   ├── Elevator.cpp
│   │   │   │   │   ├── Wrist.cpp
│   │   │   │   │   ├── Climber.cpp
│   │   │   │   │   ├── LEDController.cpp
│   │   │   │   │   ├── LED_Groups.cpp
│   │   │   │   │   ├── Turret.cpp
│   │   │   │   │   ├── TurretIntake.cpp
│   │   │   │   │   └── Turret_Shooter.cpp
│   │   │   │   └── utils/
│   │   │   │       └── POIGenerator.cpp
│   │   │   ├── include/
│   │   │   │   ├── Robot.hpp
│   │   │   │   ├── Constants.hpp      # All hardware IDs, PID gains, drive params
│   │   │   │   ├── SDSModuleType.hpp  # SDS swerve module gear ratio descriptor
│   │   │   │   ├── commands/
│   │   │   │   │   └── AutoWheelOffsets.h
│   │   │   │   ├── subsystems/
│   │   │   │   │   ├── SwerveDrive.hpp
│   │   │   │   │   ├── SwerveModule.hpp
│   │   │   │   │   ├── PoseEstimator.h
│   │   │   │   │   ├── Elevator.h
│   │   │   │   │   ├── Wrist.h
│   │   │   │   │   ├── Climber.h
│   │   │   │   │   ├── LEDController.h
│   │   │   │   │   ├── LED_Groups.h
│   │   │   │   │   ├── Turret.h
│   │   │   │   │   ├── TurretIntake.h
│   │   │   │   │   └── Turret_Shooter.h
│   │   │   │   └── utils/
│   │   │   │       ├── POIGenerator.h
│   │   │   │       └── PoseFilter.h   # Header-only Eigen pose stability filter
│   │   │   └── deploy/
│   │   │       └── pathplanner/
│   │   │           └── paths/         # PathPlanner .path JSON files
│   │   └── test/
│   │       └── cpp/
│   │           └── main.cpp
│   ├── vendordeps/                    # Vendor library JSON descriptors
│   │   ├── PathplannerLib-2025.2.7.json
│   │   ├── Phoenix6-replay-frc2025-latest.json
│   │   ├── Phoenix5-replay-5.35.1.json
│   │   ├── REVLib.json
│   │   ├── Studica-2025.0.1.json
│   │   └── WPILibNewCommands.json
│   ├── PathCalibrator/                # Python path calibration tool
│   ├── .github/workflows/             # CI workflow definitions
│   └── .Glass/                        # Glass simulation layout files
├── vendordeps/                        # Root project vendor library descriptors
│   ├── CommandsV2.json
│   └── wpilib-installation.md
├── build/                             # Gradle build outputs (generated, not committed)
├── gradle/wrapper/                    # Gradle wrapper JARs
├── build.gradle                       # Root project Gradle build config
├── settings.gradle                    # Gradle settings (project name)
├── gradlew / gradlew.bat              # Gradle wrapper scripts
├── .wpilib/                           # WPILib team/year preferences
├── .vscode/                           # VS Code C++ intellisense config
├── docs/                              # Project documentation
└── .planning/                         # GSD planning artifacts
    └── codebase/                      # Codebase map documents (this dir)
```

## Directory Purposes

**`122-Swerve-Template/src/main/cpp/subsystems/`:**
- Purpose: Implementation of all robot mechanism subsystems
- Contains: One `.cpp` per mechanism — SwerveDrive, SwerveModule, PoseEstimator, Elevator, Wrist, Climber, LEDController, LED_Groups, Turret, TurretIntake, Turret_Shooter
- Key files: `SwerveDrive.cpp` (most complex — drive, odometry, vision fusion), `SwerveModule.cpp` (per-wheel control)

**`122-Swerve-Template/src/main/include/subsystems/`:**
- Purpose: Header declarations for all subsystems — class interface, member variables, dependencies
- Contains: `.hpp` / `.h` files mirroring the `cpp/subsystems/` directory
- Key files: `SwerveDrive.hpp`, `SwerveModule.hpp`, `PoseEstimator.h`

**`122-Swerve-Template/src/main/include/utils/`:**
- Purpose: Non-subsystem utility classes and helper code
- Contains: `POIGenerator.h` (field landmark management), `PoseFilter.h` (header-only Eigen filter)
- Key files: `PoseFilter.h` — fully inlined in header, no corresponding `.cpp`

**`122-Swerve-Template/src/main/include/Constants.hpp`:**
- Purpose: Single source of truth for all compile-time configuration
- Contains: `ElectricalConstants` (CAN IDs), `DriveConstants` (speeds, offsets, controller ports), `ModuleConstants` (PID gains, gear ratios), `MathUtilNK` (utility functions)

**`122-Swerve-Template/src/main/deploy/pathplanner/paths/`:**
- Purpose: PathPlanner `.path` files deployed to robot — define autonomous trajectories
- Generated: By PathPlanner GUI tool
- Committed: Yes

**`122-Swerve-Template/vendordeps/`:**
- Purpose: WPILib vendor dependency JSON descriptors — define which third-party libraries are used and their versions
- Generated: No (manually added via WPILib VS Code extension)
- Committed: Yes

**`122-Swerve-Template/PathCalibrator/`:**
- Purpose: Python scripts for calibrating PathPlanner paths against real robot behavior
- Contains: Python scripts with `__pycache__`

**`build/`:**
- Purpose: Gradle build output — compiled objects, executables
- Generated: Yes
- Committed: No

## Key File Locations

**Entry Points:**
- `122-Swerve-Template/src/main/cpp/Robot.cpp`: `main()` at line 279 — competition robot entry point
- `src/main/cpp/Robot.cpp`: `main()` at line 78 — skeleton project entry point
- `src/test/cpp/main.cpp`: GoogleTest entry point
- `122-Swerve-Template/src/test/cpp/main.cpp`: GoogleTest entry point for template

**Configuration:**
- `122-Swerve-Template/src/main/include/Constants.hpp`: All hardware IDs, PID tuning, drive parameters
- `build.gradle`: GradleRIO build config — target platform (SystemCore), vendor library deps, test setup
- `122-Swerve-Template/vendordeps/*.json`: Per-library vendor descriptor files
- `.wpilib/wpilib_preferences.json`: Team number and WPILib year setting

**Core Logic:**
- `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp`: Drive control, odometry, vision fusion
- `122-Swerve-Template/src/main/cpp/subsystems/SwerveModule.cpp`: Per-module motor control
- `122-Swerve-Template/src/main/include/utils/PoseFilter.h`: Vision pose validation (header-only)
- `122-Swerve-Template/src/main/include/utils/POIGenerator.h`: Field POI management

**Commands:**
- `122-Swerve-Template/src/main/cpp/commands/AutoWheelOffsets.cpp`: Wheel offset calibration command
- `src/main/cpp/commands/Autos.cpp`: Skeleton auto factory

**Testing:**
- `src/test/cpp/main.cpp`: Root project test entry
- `122-Swerve-Template/src/test/cpp/main.cpp`: Template test entry (GoogleTest)

## Naming Conventions

**Files:**
- Subsystem classes: PascalCase matching class name — `SwerveDrive.cpp` / `SwerveDrive.hpp`
- Utility classes: PascalCase — `POIGenerator.h`, `PoseFilter.h`
- Some headers use `.hpp` (newer WPILib style: `SwerveDrive.hpp`, `SwerveModule.hpp`) and some use `.h` (older style: `Elevator.h`, `Wrist.h`)
- Commands: PascalCase — `AutoWheelOffsets.cpp` / `AutoWheelOffsets.h`

**Directories:**
- Lowercase: `subsystems/`, `commands/`, `utils/`
- Headers mirror source: `include/subsystems/` matches `cpp/subsystems/`

**Classes:**
- PascalCase: `SwerveDrive`, `SwerveModule`, `PoseEstimator`, `POIGenerator`
- No `I` interface prefix, no `Base` suffix except WPILib base classes (`SubsystemBase`)

**Constants:**
- `k` prefix + PascalCase for constant names: `kDriverPort`, `kMaxTranslationalVelocity`, `kFrontLeftDriveMotorID`
- Grouped in namespaces: `ElectricalConstants::`, `DriveConstants::`, `ModuleConstants::`
- Hardware member variables: `m_` prefix + PascalCase: `m_driveMotor`, `m_swerveDrive`, `m_pdh`

**Member Variables:**
- Private members use `m_` prefix: `m_driveMotor`, `m_swerveDrive`, `m_poseEstimator`
- Some older members omit prefix: `navx`, `modules`, `speeds` in `SwerveDrive.hpp`

## Where to Add New Code

**New Subsystem (mechanism):**
- Implementation: `122-Swerve-Template/src/main/cpp/subsystems/NewMechanism.cpp`
- Header: `122-Swerve-Template/src/main/include/subsystems/NewMechanism.h`
- Register: Add instance to `Robot.hpp` private section; call in `Robot::CreateRobot()` and `Robot::BindCommands()`
- Follow: `Climber.h` / `Climber.cpp` as a pattern for a simple mechanism

**New Command:**
- Implementation: `122-Swerve-Template/src/main/cpp/commands/NewCommand.cpp`
- Header: `122-Swerve-Template/src/main/include/commands/NewCommand.h`
- Use: Inherit `frc2::CommandHelper<frc2::Command, NewCommand>`; bind in `Robot::BindCommands()`

**New Utility:**
- Header-only: `122-Swerve-Template/src/main/include/utils/NewUtil.h` (if stateless/inline like `PoseFilter.h`)
- With implementation: `122-Swerve-Template/src/main/include/utils/NewUtil.h` + `122-Swerve-Template/src/main/cpp/utils/NewUtil.cpp`

**New Constants:**
- Add a new namespace to `122-Swerve-Template/src/main/include/Constants.hpp`
- Pattern: `namespace NewMechanismConstants { const int kMotorID = 50; ... }`

**New Autonomous Path:**
- Design path in PathPlanner GUI → saves to `122-Swerve-Template/src/main/deploy/pathplanner/paths/`
- Reference path name via `pathplanner::PathPlannerAuto` or `AutoBuilder`

**New Tests:**
- Add `.cpp` test files to `src/test/cpp/` or `122-Swerve-Template/src/test/cpp/`
- Use GoogleTest (`TEST`, `TEST_F`) macros; framework already configured in `build.gradle`

## Special Directories

**`.wpilib/`:**
- Purpose: WPILib team number, year, and IDE preferences
- Generated: Partially (by WPILib VS Code extension on first open)
- Committed: Yes (team number is needed for deploy)

**`.Glass/`:**
- Purpose: Glass simulation GUI window layout configuration
- Generated: Yes (by Glass tool during simulation)
- Committed: Yes (preserves useful simulation layouts)

**`build/`:**
- Purpose: Compiled object files, linked executables, Gradle task outputs
- Generated: Yes
- Committed: No (in `.gitignore`)

**`.gradle/`:**
- Purpose: Gradle build cache and metadata
- Generated: Yes
- Committed: No

**`122-Swerve-Template/PathCalibrator/__pycache__/`:**
- Purpose: Python bytecode cache for PathCalibrator scripts
- Generated: Yes
- Committed: No

---

*Structure analysis: 2026-06-22*
