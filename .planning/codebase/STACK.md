# Technology Stack

**Analysis Date:** 2026-06-22

## Languages

**Primary:**
- C++17 - All robot logic, subsystems, commands, and hardware interfaces

**Secondary:**
- Gradle (Groovy DSL) - Build system configuration (`build.gradle`, `settings.gradle`)

## Runtime

**Environment:**
- Root project: WPILib SystemCore platform (NI SystemCore coprocessor target) — `linuxsystemcore`
- Swerve template subproject: NI roboRIO (`linuxathena`) target
- Desktop simulation supported on both projects (`includeDesktopSupport = true`)

**Build Tool:**
- Gradle with Gradle wrapper (`gradlew`)
- Root plugin: `org.wpilib.GradleRIO` version `2027.0.0-alpha-6` (alpha/pre-release)
- Swerve template plugin: `edu.wpi.first.GradleRIO` version `2025.3.2` (stable)

## Frameworks

**Core:**
- WPILib — FRC robot framework providing `TimedRobot`, `SubsystemBase`, command scheduling, kinematics, odometry, simulation support
  - Root project uses alpha `2027_alpha5` WPILib year
  - Swerve template uses stable `2025` WPILib year
- Commands V2 (`commandsv2`) — WPILib command-based programming framework (vendordep: `vendordeps/CommandsV2.json`)

**Testing:**
- Google Test (GoogleTest) — C++ unit testing framework, integrated via `wpi.cpp.deps.googleTest(it)` in both projects
- WPILib HAL initialized before tests (`HAL_Initialize(500, 0)`) in `src/test/cpp/main.cpp`

**Build/Dev:**
- WPILib Simulation GUI (`wpi.sim.addGui()`) — desktop simulation with visual interface, enabled by default
- WPILib Driver Station simulation (`wpi.sim.addDriverstation()`) — available but not default
- VS Code with `vscode-wpilib` C++ IntelliSense provider (`.vscode/settings.json`)

## Key Dependencies

**Critical (Root Project):**
- WPILib `2027_alpha5` — core robot framework (alpha pre-release); sourced from local WPILib home maven (`C:\Users\Public\wpilib\2027_alpha5\maven` on Windows)
- Commands V2 (`commandsv2-cpp`) — command scheduler and button/trigger bindings

**Critical (Swerve Template — `122-Swerve-Template/`):**
- WPILib `2025` — core robot framework (stable)
- CTRE Phoenix 6 Replay `25.3.0` — motor controller and sensor API for TalonFX, CANcoder, Pigeon2 IMU (`vendordeps/Phoenix6-replay-frc2025-latest.json`)
- PathPlannerLib `2025.2.7` — autonomous path planning and following (`vendordeps/PathplannerLib-2025.2.7.json`)
- REVLib `2025.0.3` — REV Robotics motor controller support (vendordep present; not actively referenced in observed source) (`vendordeps/REVLib.json`)
- Studica `2025.0.1` — NavX gyroscope/IMU library (`studica::AHRS`) (`vendordeps/Studica-2025.0.1.json`)
- Phoenix 5 Replay `5.35.1` — legacy CTRE Phoenix 5 support (`vendordeps/Phoenix5-replay-5.35.1.json`)

## Configuration

**Robot Identity:**
- Team number: `122`, stored in `.wpilib/wpilib_preferences.json` and `122-Swerve-Template/.wpilib/wpilib_preferences.json`
- WPILib project year: `2027_alpha5` (root), `2025` (swerve template)

**Build:**
- `build.gradle` — GradleRIO plugin config, deploy targets, source sets, vendor/WPILib deps
- `settings.gradle` — Gradle plugin management, local WPILib maven repository resolution
- `vendordeps/*.json` — individual vendor library descriptors pulled at build time

**Deploy:**
- Root project deploys to SystemCore at default hostname; static files to `/home/systemcore/deploy/`
- Swerve template deploys to roboRIO; static files to `/home/lvuser/deploy/`
- PathPlanner navigation grid and settings deploy as static files (`src/main/deploy/pathplanner/`)

## Platform Requirements

**Development:**
- WPILib installation required at `C:\Users\Public\wpilib\2027_alpha5\` (Windows) or `~/wpilib/2027_alpha5/` (Linux/macOS) for root project
- VS Code with WPILib extension recommended (C++ IntelliSense configured via `vscode-wpilib`)
- Gradle wrapper included; no separate Gradle installation needed

**Production:**
- Root project: NI SystemCore coprocessor running Linux (`linuxsystemcore`)
- Swerve template: NI roboRIO running Linux (`linuxathena`)
- CAN bus network for motor controllers and sensors (CTRE CANivore named `"NKCANivore"` used for Pigeon2)

---

*Stack analysis: 2026-06-22*
