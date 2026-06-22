# External Integrations

**Analysis Date:** 2026-06-22

## Hardware Integrations (Physical Robot Devices)

**Motor Controllers — CTRE TalonFX (Falcon 500 / Kraken X60):**
- Used for all 8 swerve drive motors (4 drive, 4 steer) in `122-Swerve-Template`
- CAN IDs defined in `122-Swerve-Template/src/main/include/Constants.hpp` under `ElectricalConstants`
  - Front Left: drive=10, steer=11
  - Front Right: drive=20, steer=21
  - Back Left: drive=30, steer=31
  - Back Right: drive=40, steer=41
- SDK: `ctre::phoenix6::hardware::TalonFX` from Phoenix 6 library
- CAN bus: `"NKCANivore"` (named CANivore device)

**Absolute Encoders — CTRE CANcoder:**
- One per swerve module for steer angle measurement (IDs 12, 22, 32, 42)
- SDK: `ctre::phoenix6::hardware::CANcoder` from Phoenix 6 library
- Header: `<ctre/phoenix6/CANcoder.hpp>`

**IMU — CTRE Pigeon2:**
- CAN ID: 2, on `"NKCANivore"` CAN bus
- Used for robot heading/orientation in `SwerveDrive`
- SDK: `ctre::phoenix6::hardware::Pigeon2`
- Header: `<ctre/phoenix6/Pigeon2.hpp>`
- Simulation state: `ctre::phoenix6::sim::Pigeon2SimState`
- File: `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp`

**IMU — NavX (studica::AHRS):**
- Connected via MXP SPI port (`studica::AHRS::NavXComType::kMXP_SPI`)
- Declared but commented out in `SwerveDrive`; Pigeon2 is the active IMU
- SDK: `studica::AHRS` from Studica vendor library
- Header: `<studica/AHRS.h>`
- File: `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp`

**Gamepad/Controller:**
- Driver controller on port 0 (`OperatorConstants::kDriverControllerPort` in root project)
- Root project uses `wpi::cmd::CommandGamepad` (WPILib Commands V2 alpha API)
- Swerve template uses ports 0 (driver), 1 (operator), 2 (debug) defined in `DriveConstants`
- File (root): `src/main/include/RobotContainer.hpp`
- File (template): `122-Swerve-Template/src/main/include/Constants.hpp`

## Autonomous Path Planning

**PathPlannerLib 2025.2.7:**
- Autonomous trajectory generation and following
- Maven source: `https://3015rangerrobotics.github.io/pathplannerlib/repo`
- Used in `SwerveDrive` via `pathplanner::lib::auto::AutoBuilder` and `PPHolonomicDriveController`
- Deployed config files: `src/main/deploy/pathplanner/settings.json` and `navgrid.json`
- Headers: `<pathplanner/lib/auto/AutoBuilder.h>`, `<pathplanner/lib/config/RobotConfig.h>`, `<pathplanner/lib/controllers/PPHolonomicDriveController.h>`
- File: `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp`

## Vision / Pose Estimation

**NetworkTables-based Vision:**
- Pose estimates received over NetworkTables from an external vision coprocessor
- Table keys: `base_link_1`, `base_link_2`, `vision_stddev`, `time`
- Subscribers: `nt::DoubleArraySubscriber` for pose and standard deviation data
- Publisher: `nt::DoubleArrayPublisher` for odometry output
- SDK: `<networktables/NetworkTableInstance.h>`, `<networktables/DoubleArrayTopic.h>`
- Vision data fused into `frc::SwerveDrivePoseEstimator<4>` with custom `PoseFilter` weighting
- File: `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp`
- Implementation: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp`

## Driver Dashboard / Telemetry

**SmartDashboard:**
- Used for publishing motor and module state data during operation
- SDK: `<frc/smartdashboard/SmartDashboard.h>`

**Shuffleboard:**
- Used for structured telemetry layout (initialized via `ShuffleboardInit()`, updated via `PeriodicShuffleboard()`)
- SDK: `<frc/shuffleboard/Shuffleboard.h>`
- File: `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp`

**Field2d:**
- 2D field visualization widget for pose display on dashboard
- SDK: `<frc/smartdashboard/Field2d.h>`

**Glass (WPILib simulation GUI):**
- Config stored at `122-Swerve-Template/.Glass/glass.json`
- Used during desktop simulation sessions

## Data Storage

**Databases:** None — no persistent database
**File Storage:** Local filesystem only — deployed files on robot via GradleRIO `FileTreeArtifact`
**Caching:** None

## Authentication & Identity

**Auth Provider:** Not applicable — embedded robotics system, no user authentication

## CI/CD & Deployment

**CI Pipeline (Swerve Template only):**
- GitHub Actions — `.github/workflows/build.yml` (not present in root project)
- Triggers: push and pull request events on all branches
- Runner: `ubuntu-latest` with `wpilib/roborio-cross-ubuntu:2025-22.04` Docker container
- Build command: `./gradlew assemble`

**Robot Deployment:**
- Root project: GradleRIO deploy to SystemCore (`./gradlew deploy` or VS Code WPILib deploy)
- Swerve template: GradleRIO deploy to roboRIO
- Static deploy files (PathPlanner configs) copied to `/home/systemcore/deploy/` or `/home/lvuser/deploy/`

## Environment Configuration

**Required configuration:**
- WPILib local installation at platform-specific path (`PUBLIC\wpilib\2027_alpha5` on Windows)
- Team number set in `.wpilib/wpilib_preferences.json` (currently team 122)
- CAN bus device names configured in source (`"NKCANivore"` hardcoded in `SwerveDrive.hpp`)
- Swerve module wheel offsets configured via `frc::Preferences` (written to robot via Shuffleboard)

**No `.env` files or secret management** — embedded robotics application has no cloud credentials or API keys

## Webhooks & Callbacks

**Incoming:** None
**Outgoing:** None

---

*Integration audit: 2026-06-22*
