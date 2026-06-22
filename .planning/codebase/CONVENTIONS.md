# Coding Conventions

**Analysis Date:** 2026-06-22

## Project Structure

This repository has two C++ WPILib robot projects:

- **Root project** (`src/`): A base WPILib Command-based template targeting the newer SystemCore platform (GradleRIO `2027.0.0-alpha-6`, plugin `org.wpilib.GradleRIO`).
- **Submodule** (`122-Swerve-Template/src/`): A full competition robot targeting RoboRIO (GradleRIO `2025.3.2`, plugin `edu.wpi.first.GradleRIO`). This is the primary codebase with real logic.

Both use C++17 with WPILib's Command-based framework.

## Naming Patterns

**Files:**
- Header files use `.hpp` for classes that include template or inline code (e.g., `SwerveDrive.hpp`, `SwerveModule.hpp`, `Constants.hpp`).
- Header files use `.h` for simpler subsystem/command headers (e.g., `Climber.h`, `Elevator.h`, `Turret.h`).
- Implementation files always use `.cpp`.
- File names match the class name exactly (e.g., `SwerveDrive.hpp` contains `class SwerveDrive`).
- Underscore in file names appears when two related concepts join: `LED_Groups.h`, `Turret_Shooter.h`.

**Classes:**
- PascalCase: `SwerveDrive`, `SwerveModule`, `RobotContainer`, `ExampleSubsystem`, `POIGenerator`, `PoseFilter`, `PoseEstimator`.

**Methods (Member Functions):**
- PascalCase: `GetPose()`, `ResetHeading()`, `SetDesiredState()`, `BindCommands()`, `UpdateDashboard()`, `Periodic()`, `SimulationPeriodic()`.
- Boolean query methods use no prefix: `ExampleCondition()`, `atSetpoint()` (lowercase `a` is inconsistent — most use PascalCase).
- Factory methods follow WPILib convention: `ExampleMethodCommand()`, `GetAutonomousCommand()`.

**Variables — Member Fields:**
- WPILib convention uses `m_` prefix for private member variables: `m_driveMotor`, `m_steerEncoder`, `m_angleOffset`, `m_pigeon`, `m_poseEstimator`, `m_simTimer`.
- Some members omit the prefix: `navx`, `speeds`, `pidX`, `pidY`, `pidRot`, `hasRun`, `enable`.
- This is inconsistent — the `m_` prefix is preferred per WPILib convention.

**Variables — Local / Parameters:**
- camelCase: `driveMotorId`, `steerMotorId`, `leftXAxis`, `rightXAxis`, `driveConfig`, `steerConfig`.

**Constants:**
- Constants inside namespaces use the `k` prefix + PascalCase: `kDriverPort`, `kMaxTranslationalVelocity`, `kDriveP`, `kFrontLeftOffset`.
- String key constants use `inline constexpr std::string_view` and follow the same `k` prefix pattern: `kFrontLeftOffsetKey`.
- PID tuning constants (`kP`, `kI`, `kD`, `kS`, `kV`, `kA`) map to CTRE's naming convention.

**Namespaces:**
- Constants are grouped into domain namespaces: `ElectricalConstants`, `DriveConstants`, `ModuleConstants`, `MathUtilNK`, `OperatorConstants`.
- Command factory functions use namespaces: `autos::ExampleAuto(...)`.
- WPILib types are accessed via `wpi::cmd::` (newer template) or `frc2::` / `frc::` (submodule).

## Code Style

**Formatting:**
- No `.clang-format` file detected; formatting is done manually.
- Opening braces for class/function bodies: two styles coexist.
  - Root project: opening brace on the same line: `void Robot::RobotPeriodic() {`
  - Submodule: opening brace on a new line: `void Robot::RobotPeriodic()\n{`
- Indentation: 2 spaces (root project), 4 spaces (submodule). **Use 4 spaces in the submodule, 2 in the root project.**
- Public/private access specifiers are indented 2 spaces inside class body.

**Linting:**
- No `.clang-tidy` or other linting config detected. Rely on compiler warnings.

## Include Organization

**Order (submodule pattern):**
1. Standard library headers (`<cmath>`, `<array>`, `<string>`, `<optional>`)
2. Vendor/external library headers (`<ctre/...>`, `<frc/...>`, `<frc2/...>`, `<pathplanner/...>`, `<units/...>`, `<networktables/...>`)
3. Project-local headers (`"Constants.hpp"`, `"subsystems/SwerveDrive.hpp"`, `"utils/POIGenerator.h"`)

All `#include` directives in headers are angle-bracket form for vendor libs and quoted form for project files.

**Header Guards:**
- Use `#pragma once` (not traditional `#ifndef` guards) universally throughout the project.

## Class Design

**Subsystems:**
- All subsystems inherit from `frc2::SubsystemBase` (submodule) or `wpi::cmd::SubsystemBase` (root).
- Hardware components (motor controllers, encoders, sensors) are declared `private`.
- Public API exposes only methods — no public hardware members.
- Every subsystem overrides `Periodic()` and `SimulationPeriodic()`.

**Commands:**
- Commands inherit via the CRTP helper: `wpi::cmd::CommandHelper<wpi::cmd::Command, DerivedClass>`.
- `explicit` keyword used on single-argument constructors: `explicit ExampleCommand(ExampleSubsystem* subsystem)`.
- Subsystem raw pointers passed to commands (not references or smart pointers).
- `AddRequirements(subsystem)` called in constructor body.

**Robot Class:**
- Extends `frc::TimedRobot` (submodule) or `wpi::TimedRobot` (root).
- Robot lifecycle methods (`RobotInit`, `RobotPeriodic`, `AutonomousInit`, etc.) are all overrides.
- Hardware/subsystem initialization extracted into `CreateRobot()` private method (submodule pattern).
- Button bindings extracted into `BindCommands()` private method.

## Constants File Pattern

Constants live entirely in `src/main/include/Constants.hpp`. Never in `.cpp` files.

```cpp
namespace ElectricalConstants {
    const int kFrontLeftDriveMotorID = 10;
    // ...
}

namespace DriveConstants {
    const auto kMaxTranslationalVelocity = units::meters_per_second_t{4};
    inline constexpr std::string_view kFrontLeftOffsetKey = "kFrontLeftOffset";
    // ...
}
```

Use `inline constexpr` for string constants to avoid ODR violations. Use `const` (not `constexpr`) for non-literal types like `frc::Rotation2d` and `frc::Translation2d`.

## Lambda Usage

Lambdas are used extensively for command-based bindings:
```cpp
// Default drive command
m_swerveDrive.SetDefaultCommand(frc2::RunCommand([this] { /* axis reading */ }, {&m_swerveDrive}));

// Button binding
frc2::JoystickButton(&m_driverController, 1)
    .OnTrue(frc2::CommandPtr(frc2::InstantCommand([this] { return m_swerveDrive.ResetHeading(); })));
```

Capture `[this]` for member access. Use `[/* this */]` to suppress capture-not-used warnings when the lambda body is a placeholder.

## Simulation Support

All hardware-interacting code uses `if constexpr (frc::RobotBase::IsSimulation())` guards for simulation-specific paths. This is compile-time branching, not runtime checks.

```cpp
if constexpr (frc::RobotBase::IsSimulation()) {
    m_simTimer.Start();
}
```

## Commented-Out Code

The codebase contains significant amounts of commented-out code (disabled subsystems, alternative hardware configurations, deprecated features). This is an accepted practice for robot code where hardware configurations change season-to-season. Keep such blocks clearly marked with a reason comment (e.g., `// uncomment if there is no CANivore being utilized`).

## Error Handling

**Strategy:** None explicit. This is embedded/real-time code — exceptions are not used. The WPILib framework handles hardware errors internally. Errors are surfaced via SmartDashboard/DataLog.

**Patterns:**
- No `try`/`catch` in robot code (exception overhead unacceptable at 20ms loop rate).
- Hardware initialization errors surface through WPILib's HAL error reporting.
- `std::optional<wpi::cmd::CommandPtr>` used for autonomous command to safely handle the "no command" case.
- Python calibration utilities use `try`/`except` around file I/O.

## Logging

**Framework:** `frc::DataLogManager` (WPILib binary data log) + `frc::SmartDashboard`/`frc::Shuffleboard` for live dashboard display.

**Patterns:**
- `frc::DataLogManager::Start()` called in `RobotInit()`.
- Structured log entries created for PDP metrics: `wpi::log::DoubleLogEntry m_VoltageLog`.
- `SmartDashboard::PutNumber/PutString/PutBoolean/PutData` used throughout `Periodic()` calls for live telemetry.
- No `std::cout` or `printf` in production code (though `<iostream>` is included in some headers).

## Comments

**When to Comment:**
- All public class members and methods get a brief Javadoc-style comment block.
- Hardware alternate configurations are documented inline with `// uncomment if...`.
- TODO comments mark items needing attention: `// TODO: retune constants`.
- `NOTE:` prefix used for important architectural observations.

**Comment Style:**
```cpp
/**
 * Will be called periodically whenever the CommandScheduler runs.
 */
void Periodic() override;
```

Single-line comments use `//` with a space after.

## Python (PathCalibrator)

Python files in `122-Swerve-Template/PathCalibrator/` follow standard Python conventions:
- Type hints used: `Dict[str, Waypoint]`, `List[Any]`.
- Docstrings on module-level functions.
- `pathlib.Path` for file operations.
- `argparse` for CLI argument parsing.

---

*Convention analysis: 2026-06-22*
