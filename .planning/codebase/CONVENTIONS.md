# Coding Conventions

**Analysis Date:** 2026-06-22

## Naming Patterns

**Files:**
- Header files use `.hpp` extension: `Robot.hpp`, `ExampleSubsystem.hpp`, `Constants.hpp`
- Source files use `.cpp` extension: `Robot.cpp`, `ExampleSubsystem.cpp`
- File names match their primary class name exactly (PascalCase): `ExampleSubsystem.hpp` contains `ExampleSubsystem`
- Free-function namespace files use camelCase namespace name: `Autos.hpp` / `Autos.cpp` for namespace `autos`

**Classes:**
- PascalCase for all class names: `Robot`, `RobotContainer`, `ExampleSubsystem`, `ExampleCommand`
- Commands extend `wpi::cmd::CommandHelper<wpi::cmd::Command, DerivedClass>` — NOT `wpi::cmd::Command` directly
- Subsystems extend `wpi::cmd::SubsystemBase`
- Robot extends `wpi::TimedRobot`

**Methods:**
- PascalCase for all public methods (WPILib convention): `RobotPeriodic()`, `ExampleMethodCommand()`, `ConfigureBindings()`
- Private helper methods also PascalCase: `ConfigureBindings()`
- Override methods retain WPILib PascalCase names: `Periodic()`, `SimulationPeriodic()`

**Variables and Members:**
- Private member variables use camelCase without prefix: `autonomousCommand`, `driverController`, `subsystem`
- Constants use `k` prefix in camelCase: `kDriverControllerPort`
- `inline constexpr` used for compile-time constants

**Namespaces:**
- Constants are grouped in subsystem-specific namespaces within `Constants.hpp`: `namespace OperatorConstants { ... }`
- Free-function groups (auto factories) use lowercase namespaces: `namespace autos { ... }`
- WPILib types accessed via `wpi::cmd::` and `wpi::` prefixes

## Header/Source Separation (WPILib Pattern)

**Headers:** `src/main/include/` — class declarations only
- Subsystem headers: `src/main/include/subsystems/ExampleSubsystem.hpp`
- Command headers: `src/main/include/commands/ExampleCommand.hpp`
- Namespace function headers: `src/main/include/commands/Autos.hpp`
- Robot-wide: `src/main/include/Robot.hpp`, `src/main/include/RobotContainer.hpp`
- Constants: `src/main/include/Constants.hpp` (all `inline constexpr` — no `.cpp` needed)

**Sources:** `src/main/cpp/` — method implementations only
- Mirror subdirectory structure of `include/`
- Include their own header first: `#include "Robot.hpp"` then WPILib headers

**Include style:**
- Project headers use double-quotes: `#include "RobotContainer.hpp"`
- WPILib and system headers use angle brackets: `#include <optional>`
- All headers use `#pragma once` (not include guards)

## WPILib Inheritance Patterns

**Robot class** (`src/main/cpp/Robot.cpp`, `src/main/include/Robot.hpp`):
- Inherits `wpi::TimedRobot`
- Override all lifecycle methods: `RobotPeriodic`, `DisabledInit`, `DisabledPeriodic`, `AutonomousInit`, `AutonomousPeriodic`, `TeleopInit`, `TeleopPeriodic`, `UtilityPeriodic`, `SimulationInit`, `SimulationPeriodic`
- Entry point guarded by `#ifndef RUNNING_WPILIB_TESTS` to allow test compilation without `main()` conflict
- Uses `std::optional<wpi::cmd::CommandPtr>` for autonomous command to avoid undefined behavior

**Subsystem classes** (`src/main/include/subsystems/`, `src/main/cpp/subsystems/`):
- Inherit `wpi::cmd::SubsystemBase`
- Override `Periodic()` for 20ms loop logic
- Override `SimulationPeriodic()` for simulation-specific updates
- Expose command factory methods (return `wpi::cmd::CommandPtr`) as public interface
- Hardware components (motors, sensors) declared `private`; accessed only via public methods

**Command classes** (`src/main/include/commands/`, `src/main/cpp/commands/`):
- Inherit `wpi::cmd::CommandHelper<wpi::cmd::Command, DerivedClass>` — CRTP pattern required for `.ToPtr()` decorator to work
- Receive subsystem pointer via `explicit` constructor parameter
- Store subsystem pointer as private member

**Auto factories** (`src/main/include/commands/Autos.hpp`):
- Free functions in a namespace, not a class
- Return `wpi::cmd::CommandPtr`
- Compose commands using `wpi::cmd::Sequence()`, `wpi::cmd::Parallel()`, etc.

## RobotContainer Pattern

`src/main/include/RobotContainer.hpp` / `src/main/cpp/RobotContainer.cpp`:
- Owns all subsystem instances as private value members (not pointers): `ExampleSubsystem subsystem;`
- Owns controller instances as private value members with inline initialization from constants: `wpi::cmd::CommandGamepad driverController{OperatorConstants::kDriverControllerPort};`
- `ConfigureBindings()` is a private method called from constructor
- Trigger bindings use lambda captures of `this` or subsystem pointer
- `GetAutonomousCommand()` is the only public method besides the constructor

## Code Style

**Formatting:**
- 2-space indentation throughout
- No trailing spaces
- Opening brace on same line as declaration: `class Robot : public wpi::TimedRobot {`
- Method body on same line for empty bodies: `void DisabledInit() {}`, `Robot::Robot() {}`
- Single blank line between method definitions in `.cpp` files

**Comments:**
- Copyright header on every file (WPILib BSD license block)
- Doxygen-style `/** ... */` block comments on public methods in headers
- Inline `//` comments in `.cpp` for implementation guidance
- Template/placeholder comments use `/* this */` syntax within lambdas: `[/* this */] { /* one-time action */ }`

## Error Handling

**Strategy:** No exception-based error handling in this codebase. WPILib robot code runs on a real-time loop — exceptions are generally avoided.

**Patterns:**
- `std::optional<wpi::cmd::CommandPtr>` used to safely represent absent autonomous command, with `if (autonomousCommand)` guard before use
- HAL initialization (`HAL_Initialize`) called before test runner in `src/test/cpp/main.cpp` — required for WPILib simulation to function

## Module Design

**Subsystem ownership:** `RobotContainer` owns subsystems by value. Commands receive raw pointers to subsystems.

**Scheduler:** `wpi::cmd::CommandScheduler::GetInstance().Run()` called once per `RobotPeriodic()` — single global scheduler drives all command execution.

---

*Convention analysis: 2026-06-22*
