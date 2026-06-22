# External Integrations

**Analysis Date:** 2026-06-22

## Vendor Dependencies

**CommandsV2 (`vendordeps/CommandsV2.json`):**
- Name: Commands V2
- Version: 1.0.0 (WPILib-bundled, resolves to 2027.0.0-alpha-6)
- wpilibYear: `2027_alpha5`
- Maven group: `org.wpilib.commandsv2`
- C++ artifact: `commandsv2-cpp` (shared library)
- Supported platforms: `linuxsystemcore`, `linuxathena`, `linuxarm32`, `linuxarm64`, `windowsx86-64`, `windowsx86`, `linuxx86-64`, `osxuniversal`
- No external `mavenUrls` — resolves entirely from local WPILib maven at `C:\Users\Public\wpilib\2027_alpha5\maven\`
- Conflict: cannot coexist with Commands V3 (`CommandsV3.json`, UUID `4decdc05-a056-46cf-9561-39449bbb01306`)

No other vendordeps are present. Third-party motor controller or sensor vendor libraries (e.g., CTRE Phoenix, REV SparkMax, Kauailabs NavX) are not yet installed.

## FRC Hardware Abstraction Layer (HAL)

**Library:** `hal-cpp-2027.0.0-alpha-6`

**Purpose:** Low-level hardware communication layer between robot software and NI SystemCore hardware. All WPILib hardware classes (motors, sensors, I/O) use HAL internally.

**Direct usage in this project:**
- `src/test/cpp/main.cpp` calls `HAL_Initialize(500, 0)` before running GoogleTest — required to allow WPILib classes to function in test context
- Header: `wpi/hal/HAL.h`

**HAL capabilities available (not yet used in project):**
- PWM output control (motor controllers via PWM)
- Digital I/O (limit switches, beam breaks)
- Analog I/O (potentiometers, distance sensors)
- CAN bus communication (motor controllers, IMUs)
- Serial / SPI / I2C communication
- Encoder (quadrature counter) input
- Relay output control
- DMA support

## Network Protocols

### FRC Driver Station Protocol
- **Direction:** Inbound from FRC Driver Station software (DS) over USB or Ethernet
- **What it provides:** Enable/disable signals, robot mode (teleop/auto/test/disabled), alliance station, match time, joystick/gamepad data
- **Integration point:** `wpi::TimedRobot` (via HAL) receives mode transitions and calls the appropriate `*Init()` / `*Periodic()` methods on `Robot` (`src/main/cpp/Robot.cpp`)
- **Gamepad access:** `wpi::cmd::CommandGamepad driverController{OperatorConstants::kDriverControllerPort}` in `src/main/include/RobotContainer.hpp` — port 0 maps to DS joystick slot 0

### NetworkTables 4 (NT4)
- **Library:** `ntcore-cpp-2027.0.0-alpha-6`
- **Protocol:** WebSocket-based publish/subscribe over port 5810 (robot) ↔ DS/Shuffleboard/Elastic
- **Direction:** Bidirectional — robot publishes telemetry; DS/dashboard can push values back
- **Status in this project:** Not yet directly used in project source code, but available through WPILib. `SmartDashboard` (from `wpilibc-cpp`) uses NT4 internally.
- **Common usage pattern to add:**
  ```cpp
  #include "frc/smartdashboard/SmartDashboard.h"
  frc::SmartDashboard::PutNumber("key", value);
  ```

### WPILib Simulation Protocol
- **Simulation GUI** (`wpi.sim.addGui().defaultEnabled = true`): enabled by default for desktop runs
- **Driver Station Sim** (`wpi.sim.addDriverstation()`): available but not enabled by default
- Desktop simulation target: `windowsx86-64` — uses MSVC (`cl.exe`) from Visual Studio 2022

## Robot Hardware Target

**Platform:** NI SystemCore (replaces roboRIO starting 2027 season)
- Architecture: aarch64 (ARM 64-bit)
- OS: Linux Debian Bookworm
- GradleRIO target type: `SystemCore` (`getTargetTypeClass('SystemCore')`)
- Default hostname: set by `useDefaultSystemcoreHostName()` (team-number-based mDNS)
- Team number: 122 (`.wpilib/wpilib_preferences.json`)
- Deploy path on robot: artifact at default WPILib location; static files at `/home/systemcore/deploy/`
- `deleteOldFiles = false` for static deploy directory (safe default — won't remove files present on robot but absent locally)

**Cross-compilation toolchain:**
- Compiler: `aarch64-bookworm-linux-gnu-g++.exe` at `C:\Users\Public\wpilib\2027_alpha5\systemcore\bin\`
- Sysroot: `C:\Users\Public\wpilib\2027_alpha5\systemcore\aarch64-linux-gnu\sysroot\`
- C++ standard library: GCC 12 libstdc++ for aarch64

## Camera / Vision

**Libraries available (not yet used in project source):**
- `cameraserver-cpp-2027.0.0-alpha-6` — `CameraServer` API for USB/IP cameras
- `cscore-cpp-2027.0.0-alpha-6` — underlying camera pipeline
- `apriltag-cpp-2027.0.0-alpha-6` — AprilTag pose detection
- `opencv-cpp-2027-4.13.0-3` — OpenCV 4.13.0 for custom vision processing

## Data Logging

**Library available (not yet used in project source):**
- `datalog-cpp-2027.0.0-alpha-6` — WPILib DataLog writes structured `.wpilog` files to robot filesystem for post-match replay and analysis via WPILib DataLogTool

## Build / CI

**Local build only** — no CI/CD pipeline detected. No GitHub Actions, no Jenkins, no remote artifact publishing configured.

**Build commands:**
```bash
./gradlew build               # Compile all targets (SystemCore + Desktop)
./gradlew deploy              # Cross-compile and deploy to SystemCore over network
./gradlew simulateJavaRelease # Wrong — use:
./gradlew simulateCppRelease  # Run desktop simulation (Windows)
./gradlew test                # Run GoogleTest unit tests on desktop
```

## Static Deploy Files

**Location:** `src/main/deploy/`
**Deployed to:** `/home/systemcore/deploy/` on robot
**Current contents:** `example.txt` (placeholder)
**Purpose:** Configuration files, autonomous paths, calibration data — any files the robot program reads at runtime from the filesystem

## Environment Configuration

No `.env` files or environment variables used for configuration. All robot configuration is:
- Team number: `.wpilib/wpilib_preferences.json`
- Constants: `src/main/include/Constants.hpp` (compile-time `constexpr`)
- Deploy/debug mode: Gradle command-line flags (`-Pdebug`)

---

*Integration audit: 2026-06-22*
