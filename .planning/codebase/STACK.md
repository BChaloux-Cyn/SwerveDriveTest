# Technology Stack

**Analysis Date:** 2026-06-22

## Languages

**Primary:**
- C++23 - All robot program source code (`src/main/cpp/`, `src/main/include/`)

**Secondary:**
- Groovy (Gradle DSL) - Build system configuration (`build.gradle`, `settings.gradle`)

## Runtime

**Environment:**
- Target (deploy): Linux aarch64 (ARM 64-bit) — NI SystemCore running Debian Bookworm
- Development/Simulation: Windows x86-64 (desktop simulation)
- Cross-compile toolchain: `aarch64-bookworm-linux-gnu-g++` from WPILib installation at `C:\Users\Public\wpilib\2027_alpha5\systemcore\bin\`
- System sysroot: `C:\Users\Public\wpilib\2027_alpha5\systemcore\aarch64-linux-gnu\sysroot\` (GCC 12, C++ stdlib)

**C++ Standard:** C++23 (`-std=c++23` on GCC, `/std:c++23preview` on MSVC)

**Package Manager:**
- Gradle 9.4.1 with GradleRIO plugin
- Maven local repository: `C:\Users\Public\wpilib\2027_alpha5\maven\`
- User Gradle cache: `C:\Users\122\.gradle\caches\9.4.1\transforms\`
- Lockfile: `gradle/wrapper/gradle-wrapper.properties`

## Frameworks

**Core — FRC Robot Framework:**
- WPILib 2027.0.0-alpha-6 (`projectYear: 2027_alpha5`)
  - GradleRIO plugin version: `2027.0.0-alpha-6` (`build.gradle` line 4)
  - Provides the full FRC robot programming framework
  - Docs: https://docs.wpilib.org/en/2027/

**Command-Based Framework:**
- Commands V2 (vendordep `vendordeps/CommandsV2.json`)
  - Version: `1.0.0` / artifact version: `wpilib` (resolves to 2027.0.0-alpha-6)
  - C++ artifact: `org.wpilib.commandsv2:commandsv2-cpp` (shared library)
  - UUID: `111e20f7-815e-48f8-9dd6-e675ce75b266`
  - Conflicts with Commands V3 (cannot coexist)
  - Header cache: `commandsv2-cpp-2027.0.0-alpha-6-headers\`

**Testing:**
- GoogleTest 2027.0.0-alpha-6 — unit test framework for desktop (`src/test/cpp/`)
  - Included via `wpi.cpp.deps.googleTest(it)` in `build.gradle`
  - Header cache: `googletest-cpp-2027.0.0-alpha-6-headers\`

**Build/Dev:**
- GradleRIO `2027.0.0-alpha-6` — FRC-specific Gradle plugin (deploy, simulation, vendordep resolution)
- Gradle 9.4.1 — build system (`gradle/wrapper/gradle-wrapper.properties`)
- VS Code with WPILib extension — IDE (`C_Cpp.default.configurationProvider: vscode-wpilib`, `.vscode/settings.json`)
- WPILib Simulation GUI (`wpi.sim.addGui().defaultEnabled = true`, `build.gradle` line 50)
- WPILib Driver Station sim (`wpi.sim.addDriverstation()`, `build.gradle` line 52)

## Key WPILib Libraries (all version 2027.0.0-alpha-6)

All headers resolve from Gradle transform caches under `C:\Users\122\.gradle\caches\9.4.1\transforms\`.

| Library | Artifact | Key Classes / Purpose |
|---------|----------|-----------------------|
| **commandsv2-cpp** | `commandsv2-cpp-2027.0.0-alpha-6` | `wpi::cmd::CommandScheduler`, `wpi::cmd::SubsystemBase`, `wpi::cmd::Command`, `wpi::cmd::CommandHelper<>`, `wpi::cmd::CommandPtr`, `wpi::cmd::Trigger`, `wpi::cmd::CommandGamepad`, `wpi::cmd::Sequence()`, `wpi::cmd::RunOnce()` |
| **wpilibc-cpp** | `wpilibc-cpp-2027.0.0-alpha-6` | `wpi::TimedRobot`, `wpi::StartRobot<>()`, `SmartDashboard`, `SendableChooser`, motor controllers, sensors, `DigitalInput`, `AnalogInput`, `PWMMotorController`, `Encoder`, `PIDController`, `DifferentialDrive`, `MecanumDrive`, `SwerveModuleState`, `SwerveDriveKinematics`, `SwerveDriveOdometry` |
| **hal-cpp** | `hal-cpp-2027.0.0-alpha-6` | HAL (Hardware Abstraction Layer) — low-level hardware access; `HAL_Initialize()` required before tests (`src/test/cpp/main.cpp`) |
| **ntcore-cpp** | `ntcore-cpp-2027.0.0-alpha-6` | NetworkTables 4 — `nt::NetworkTableInstance`, `nt::NetworkTable`, publishers/subscribers for DS/dashboard communication |
| **wpimath-cpp** | `wpimath-cpp-2027.0.0-alpha-6` | Kinematics, odometry, geometry (`Translation2d`, `Rotation2d`, `Pose2d`, `ChassisSpeeds`), trajectory generation, state-space control, PID, feedforward |
| **datalog-cpp** | `datalog-cpp-2027.0.0-alpha-6` | WPILib DataLog — onboard structured logging to `.wpilog` files |
| **wpinet-cpp** | `wpinet-cpp-2027.0.0-alpha-6` | Networking utilities — HTTP, WebSockets, used internally by ntcore/cscore |
| **wpiutil-cpp** | `wpiutil-cpp-2027.0.0-alpha-6` | Shared utilities — `wpi::StringRef`, `wpi::span`, JSON, timestamp, thread utilities |
| **cameraserver-cpp** | `cameraserver-cpp-2027.0.0-alpha-6` | `CameraServer` — USB/IP camera streaming to Driver Station / Shuffleboard |
| **cscore-cpp** | `cscore-cpp-2027.0.0-alpha-6` | Camera source/sink pipeline underlying CameraServer |
| **apriltag-cpp** | `apriltag-cpp-2027.0.0-alpha-6` | AprilTag detection for vision pose estimation |
| **opencv-cpp** | `opencv-cpp-2027-4.13.0-3` | OpenCV 4.13.0 — computer vision (used by cscore/apriltag) |

## Key WPILib Classes Used Directly in This Project

| Class / Function | Header | Role in Project |
|-----------------|--------|-----------------|
| `wpi::TimedRobot` | `wpi/framework/TimedRobot.hpp` | Base class for `Robot` — provides 20ms periodic loop and mode callbacks |
| `wpi::StartRobot<Robot>()` | (wpilib internal) | Entry point called from `main()` in `src/main/cpp/Robot.cpp` |
| `wpi::cmd::CommandScheduler` | `wpi/commands2/CommandScheduler.hpp` | Singleton scheduler; `GetInstance().Run()` called every 20ms in `RobotPeriodic()` |
| `wpi::cmd::CommandPtr` | `wpi/commands2/CommandPtr.hpp` | Owning handle for a command; returned from factories and subsystem methods |
| `wpi::cmd::Command` | `wpi/commands2/Command.hpp` | Abstract base for all commands |
| `wpi::cmd::CommandHelper<>` | `wpi/commands2/CommandHelper.hpp` | CRTP wrapper required for command decorator methods to work |
| `wpi::cmd::SubsystemBase` | `wpi/commands2/SubsystemBase.hpp` | Base class for all subsystems; provides `Periodic()`, `SimulationPeriodic()`, `RunOnce()` |
| `wpi::cmd::Trigger` | `wpi/commands2/button/Trigger.hpp` | Condition-based command scheduling; `.OnTrue()`, `.WhileTrue()` bindings |
| `wpi::cmd::CommandGamepad` | `wpi/commands2/button/CommandGamepad.hpp` | Gamepad controller with trigger-returning button methods (e.g., `.EastFace()`) |
| `wpi::cmd::Sequence()` | `wpi/commands2/Commands.hpp` | Command composition — runs commands sequentially |
| `HAL_Initialize()` | `wpi/hal/HAL.h` | HAL init required in test harness (`src/test/cpp/main.cpp`) |

## Compiler Flags (SystemCore / Release)

```
-std=c++23 -Wall -Wextra -Wformat=2 -pedantic -Wno-psabi
-Wno-unused-parameter -Wno-error=deprecated-enum-enum-conversion
-fPIC -pthread -D__FIRST_SYSTEMCORE__=1 -O2
```

Preprocessor macros:
- `__FIRST_SYSTEMCORE__=1` — defined for all SystemCore builds
- `RUNNING_WPILIB_TESTS` — defined only when building the test suite (suppresses `main()` in `Robot.cpp`)

## Configuration

**Team Number:** 122 (set in `.wpilib/wpilib_preferences.json`)

**WPILib Installation:**
- Windows: `C:\Users\Public\wpilib\2027_alpha5\`
- macOS/Linux: `~/wpilib/2027_alpha5/`
- Folder name is `2027_alpha5` even for alpha-6 release (by installer design)

**Build:**
- `build.gradle` — project build definition, targets, vendordep wiring
- `settings.gradle` — plugin management, Maven repo pointing to local WPILib maven
- `gradle/wrapper/gradle-wrapper.properties` — pins Gradle 9.4.1

**Desktop Simulation:**
- `includeDesktopSupport = true` in `build.gradle` line 44 — enables desktop (Windows) target
- `wpi.cpp.debugSimulation = false` — simulation runs in release mode by default
- SimGUI enabled by default; Driver Station sim available but not default

## Platform Requirements

**Development (Windows):**
- Visual Studio 2022 Community (MSVC 14.41, found at `C:\Program Files\Microsoft Visual Studio\2022\Community\`)
- WPILib 2027.0.0-alpha-6 installer (places files at `C:\Users\Public\wpilib\2027_alpha5\`)
- VS Code with WPILib extension
- Java 11+ (required by Gradle, not by robot code)
- OS: Windows 10/11 x64

**Production (Deploy Target):**
- NI SystemCore (replaces roboRIO) — ARM64 Linux (Debian Bookworm, aarch64)
- Connected via USB or default hostname (set by `useDefaultSystemcoreHostName()`)
- Deployed artifact: `build/exe/wpilibUserProgram/linuxsystemcore/release/wpilibUserProgram`
- Static files deployed to `/home/systemcore/deploy/` on target

---

*Stack analysis: 2026-06-22*
