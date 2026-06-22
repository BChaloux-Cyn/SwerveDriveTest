# Codebase Concerns

**Analysis Date:** 2026-06-22

## Tech Debt

**Dual-robot project with divergent structure:**
- Issue: The repo contains two parallel robot programs — a WPILib2025 skeleton in `src/` (root project) and a full-featured FRC swerve drive in the `122-Swerve-Template/` Git submodule. The root `src/` project uses a different command framework (`wpi::cmd`) than the submodule (`frc2::cmd`), making them incompatible codebases. It is unclear which is the authoritative target for development.
- Files: `src/main/cpp/Robot.cpp`, `122-Swerve-Template/src/main/cpp/Robot.cpp`
- Impact: Confusion about which codebase to extend; build artifacts in root `build/` suggest the root skeleton is what compiles on this machine, while all real robot logic lives in the submodule.
- Fix approach: Decide on one canonical project and eliminate or clearly document the other.

**Submodule pinned to a non-main branch:**
- Issue: `.gitmodules` pins `122-Swerve-Template` to `DEV/brennan/system_core_update`, a developer feature branch rather than a stable main branch. This means upstream merges and releases on `main` are not tracked, and the submodule can silently drift.
- Files: `.gitmodules`
- Impact: Risk of consuming an unstable or incomplete branch as the foundation. Harder to receive upstream bug fixes.
- Fix approach: Pin to a stable release tag or `main` branch; update `.gitmodules` and commit a locked SHA.

**Dead/commented-out autonomous alignment code:**
- Issue: Approximately 40 lines of a `scoreClosest` on-the-fly PathPlanner path are commented out in `Robot.cpp`. The `scoreClosest` command in the active binding (`BindCommands()`) at line 251 points to a `scoreClosest` field that is initialized as an empty `frc2::InstantCommand`, making button 3 effectively a no-op.
- Files: `122-Swerve-Template/src/main/cpp/Robot.cpp` lines 153–191, 250–254
- Impact: Button 3 on driver controller silently does nothing; future developers may not realize the feature was intended.
- Fix approach: Restore the commented code as a proper command class, or remove the dead code and button binding.

**NavX gyro is instantiated but unused:**
- Issue: `SwerveDrive.hpp` declares `studica::AHRS navx{...}` and `SwerveDrive.cpp` calls `navx.Reset()` in the constructor, but all heading reads use `m_pigeon`. Every call site that originally used NavX is commented out.
- Files: `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp` line 113, `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 45, 247–251
- Impact: Unnecessary hardware initialization overhead; potential confusion about which IMU is in use; NavX occupies an MXP SPI slot.
- Fix approach: Remove the `navx` member and all commented NavX references, or restore its use if redundancy is desired.

**Wheel offset system uses SmartDashboard instead of Preferences:**
- Issue: `SetOffsets()` reads wheel calibration degrees from SmartDashboard via `GetNumber()` with hardcoded defaults (e.g., `-66` for FrontRight). SmartDashboard values are not guaranteed persistent across reboots unless `SetPersistent()` is called, which is done immediately after the read — meaning the first boot after a flash uses hardcoded fallbacks until a manual recalibration is run.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 511–535
- Impact: On a fresh roboRIO flash, all wheel offsets default to hardcoded values that may not match the physical robot, causing immediate swerve misalignment.
- Fix approach: Use `frc::Preferences` (already imported in `Constants.hpp`) with `InitDouble`/`GetDouble` for persistent, flash-safe calibration storage.

**`SwerveDriveKinematics` initialized twice in constructor:**
- Issue: In `SwerveDrive.cpp`, `kSwerveKinematics` is first initialized in the member initializer list (line 22), then immediately re-assigned in the constructor body (lines 36–38) with the same values.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 22–38
- Impact: Minor inefficiency; double construction of a non-trivial object. Indicates incomplete refactoring.
- Fix approach: Remove the redundant re-assignment in the constructor body.

**Commented-out subsystems in Robot (Elevator, Wrist, Climber, LED):**
- Issue: `Robot.hpp` includes and declares members for `Elevator`, `Wrist`, `Climber`, and `LEDController`, but all are commented out. The submodule still contains all these subsystem implementations, suggesting the robot platform has these mechanisms but they are not wired into the robot program.
- Files: `122-Swerve-Template/src/main/include/Robot.hpp` lines 75–85, `122-Swerve-Template/src/main/cpp/Robot.cpp` lines 50–58, 94–111
- Impact: Mechanisms exist in hardware but are not software-controlled; re-enabling requires care around initialization order and safety interlocks.
- Fix approach: Document which season/game this configuration is for; restore subsystems per competition requirements with proper safety checks.

**`WeightedDriving` marked deprecated but still in public API:**
- Issue: `WeightedDriving()` in `SwerveDrive` is flagged with a "Warning, this is unfinished (and also deprecated)" comment and annotated `// DEPRECATED` in the header, yet remains as a public method with full implementation.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` line 415, `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp` line 107
- Impact: Future developers may call a broken API; the method contains hardcoded magic numbers and incomplete tuning.
- Fix approach: Remove the method entirely or mark with `[[deprecated("Use Drive() directly")]]` and move to a separate utility file.

## Known Bugs

**`AutoWheelOffsets` reads wrong SmartDashboard key format:**
- Symptoms: `AutoWheelOffsets::Execute()` reads `"Module 1/ CANCoder Angle"` (space before CANCoder), but `SwerveModule::Periodic()` publishes `"Module 1/  CANCoder Angle"` (two-space prefix before CANCoder due to the string `" CANCoder Angle"`). These keys do not match.
- Files: `122-Swerve-Template/src/main/cpp/commands/AutoWheelOffsets.cpp` lines 45, 50, 55, 60; `122-Swerve-Template/src/main/cpp/subsystems/SwerveModule.cpp` lines 117–118
- Trigger: Running the AutoWheelOffsets command; it reads 0.0 for all modules and sets all offsets to zero, clearing calibration.
- Workaround: Manually enter offsets on SmartDashboard.

**`PoseEstimator::UpdateMeasurement()` unsafe array access:**
- Symptoms: `robotPoseValue.value.at(0)`, `at(1)`, and `at(2)` are accessed on line 40–42 without checking `robotPoseValue.value.size() > 0` first. If the ROS2Bridge robot pose topic has not published yet, this throws `std::out_of_range`.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/PoseEstimator.cpp` lines 40–42
- Trigger: Vision/ROS2 integration enabled but robot pose topic empty on startup.
- Workaround: Disable vision on startup (`useVision = false`) until ROS2 bridge is confirmed publishing.

**`SetReference()` uses `|` (bitwise OR) instead of `||` (logical OR):**
- Symptoms: In `SwerveDrive::SetReference()`, the condition `(!pidX.AtSetpoint() && !pidY.AtSetpoint()) | !hasRun` uses bitwise OR. This technically works on `bool` values in C++ but is non-idiomatic, error-prone on refactor, and may not short-circuit as expected.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` line 316
- Trigger: Called during auto alignment routines.
- Workaround: None needed currently, but fix before adding complex logic around this condition.

**`PoseFilter::lastTimestamp_` is never updated:**
- Symptoms: `PoseFilter::IsPoseValid()` reads `lastTimestamp_` (line 24) to reject duplicate timestamps, but `lastTimestamp_` is never set after the check — it always remains `std::nullopt`. Duplicate-timestamp rejection never fires.
- Files: `122-Swerve-Template/src/main/include/utils/PoseFilter.h` lines 24–26, 91
- Trigger: Any call to `IsPoseValid()`.
- Workaround: The position and rotation tolerances still filter bad poses, so the filter is partially functional.

**`Elevator::InterpolatePWL()` has no explicit return for empty range:**
- Symptoms: If `count` is 0 or 1, the for loop never executes and the function falls off the end without returning a value — undefined behavior.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/Elevator.cpp` line 521–535
- Trigger: Called with malformed constant arrays.
- Workaround: `ElevatorConstants` arrays are statically defined and non-empty, so this is latent rather than triggered.

## Security Considerations

**NetworkTable server started on robot:**
- Risk: `SwerveDrive.cpp` calls `networkTableInst.StartServer()` in the constructor. On a competition robot, starting a raw NT4 server exposes the robot to any device on the field network; accepted practice in FRC but should be removed for any non-FRC deployment.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` line 48
- Current mitigation: FRC field network is physically isolated.
- Recommendations: No action needed for FRC use; document if reused outside competition context.

**SmartDashboard used to store persistent robot configuration:**
- Risk: Wheel offsets and POI locations are stored in SmartDashboard persistent entries. Any connected driver station client can modify these values live, inadvertently miscalibrating the robot.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 511–535, `122-Swerve-Template/src/main/cpp/utils/POIGenerator.cpp` lines 37–43
- Current mitigation: None.
- Recommendations: Move safety-critical calibration to `frc::Preferences` and restrict dashboard access during competition.

## Performance Bottlenecks

**`SwerveDrive::Drive()` reads SmartDashboard for acceleration limit every call:**
- Problem: `Drive()` calls `frc::SmartDashboard::GetNumber("drive/accelLim", 2.0)` and `GetNumber("drive/vx")` / `GetNumber("drive/vy")` every 20 ms loop. SmartDashboard reads involve network table lookups and are not zero-cost.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 181–183
- Cause: Tuning values left in SmartDashboard reads in production code.
- Improvement path: Cache the acceleration limit as a member variable updated only when the dashboard value changes; store previous vx/vy as member fields rather than reading them back from the dashboard.

**`SwerveModule::Periodic()` publishes 4 SmartDashboard values per module (16 total per cycle):**
- Problem: Each of 4 modules publishes angle, CANCoder angle, velocity, and rotations to SmartDashboard every 20 ms, totaling 16 NT4 publishes per loop tick.
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveModule.cpp` lines 113–124
- Cause: Debug telemetry left enabled in production code.
- Improvement path: Gate all telemetry behind a compile-time or NT-toggled debug flag.

## Fragile Areas

**Vision pose fusion relies on ROS2 bridge being available:**
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 330–382, `122-Swerve-Template/src/main/include/subsystems/SwerveDrive.hpp`
- Why fragile: `useVision` defaults to `true`. If the ROS2 coprocessor is offline or hasn't published yet, `UpdatePoseEstimate()` runs but `baseLink1Subscribe.GetAtomic()` returns empty arrays — this is handled safely. However, `PoseEstimator.cpp` (the object pose estimator) does not guard `robotPoseSubscribe.GetAtomic().value.at(0)` access.
- Safe modification: Always check `.value.size() > 0` before indexing any NT subscriber result; add a `useVision` guard in `PoseEstimator::UpdateMeasurement()`.
- Test coverage: None — no unit tests exist for vision pipeline.

**Elevator subsystem height sensing uses complex multi-sensor fusion:**
- Files: `122-Swerve-Template/src/main/cpp/subsystems/Elevator.cpp` lines 287–366, `122-Swerve-Template/src/main/include/subsystems/Elevator.h`
- Why fragile: Height is fused from two SparkMax encoders, a CANdi Hall sensor, and a proximity sensor. The auto-calibration logic in `AutoCalibrateHeight()` silently resets `m_heightCorrection` on calibration failure. The emergency tolerance check for encoder divergence is commented out (`lines 242–247`), meaning a broken chain/belt will not be caught.
- Safe modification: Re-enable the emergency encoder divergence check before operating on a physical robot; add a maximum voltage clamp as a secondary safety.
- Test coverage: None.

**`POIGenerator::GetClosestPOI()` returns `(0,0,0)` silently when no POIs exist:**
- Files: `122-Swerve-Template/src/main/cpp/utils/POIGenerator.cpp` line 75–78
- Why fragile: Callers cannot distinguish "no POIs loaded" from a valid POI at field origin (0,0). The commented-out `scoreClosest` command in `Robot.cpp` relies on this return value for path planning.
- Safe modification: Return `std::optional<frc::Pose2d>` and update callers to handle the empty case explicitly.
- Test coverage: None.

## Scaling Limits

**Hard limit of 4 swerve modules:**
- Current capacity: `SwerveDrive` uses `std::array<SwerveModule, 4>` and all kinematics are templated on `4U`.
- Limit: Changing module count requires updating array size, kinematics template parameter, and all index-based loops throughout `SwerveDrive.cpp`.
- Scaling path: Parameterize module count as a compile-time constant; prefer ranged-for loops over index loops.

## Dependencies at Risk

**Submodule from a team-specific development branch:**
- Risk: `122-Swerve-Template` is sourced from `NASAKnights/122-Swerve-Template` at `DEV/brennan/system_core_update`. This branch may be force-pushed, rebased, or deleted at any time.
- Impact: `git submodule update` could fail or silently pick up incompatible code.
- Migration plan: Pin to a stable tag; maintain a fork if long-term stability is needed.

**WPILib 2025.3.2 (GradleRIO) with CTRE Phoenix6 and REV Spark:**
- Risk: FRC vendor dependencies (CTRE Phoenix6, REV SparkMax, PathPlanner, StudicaLib) must be updated in lockstep with each WPILib season release. Mixing versions causes runtime CANbus communication failures.
- Impact: An incomplete vendor dependency update breaks all motor/sensor communication.
- Migration plan: Update all vendordeps together using the WPILib VS Code extension or `./gradlew vendorDependencies` at the start of each season.

## Missing Critical Features

**No unit or integration tests:**
- Problem: The only test file is `src/test/cpp/main.cpp` (root project, 10 lines — Google Test boilerplate with no test cases) and `122-Swerve-Template/src/test/cpp/main.cpp` (identical). No subsystem, command, or utility has any test coverage.
- Blocks: Regression detection for kinematics, PID tuning, and pose estimation logic.

**Autonomous start pose is never set:**
- Problem: `Robot::AutonomousInit()` calls `m_swerveDrive.ResetPose(autoStartPose)` but `autoStartPose` is declared as `frc::Pose2d autoStartPose` in `Robot.hpp` with no initialization — it defaults to `(0,0,0°)`. PathPlanner autos that assume a specific starting pose will mislocalize from the start.
- Files: `122-Swerve-Template/src/main/include/Robot.hpp` line 133, `122-Swerve-Template/src/main/cpp/Robot.cpp` line 78
- Blocks: Multi-path autonomous routines requiring accurate initial pose.

**`SetFast()` and `SetSlow()` are empty stubs:**
- Problem: `SwerveDrive::SetFast()` and `SwerveDrive::SetSlow()` have empty bodies. No speed-scaling mechanism exists for "slow mode" (e.g., when elevator is raised).
- Files: `122-Swerve-Template/src/main/cpp/subsystems/SwerveDrive.cpp` lines 243–245
- Blocks: Safe operation at height — a raised elevator raises the center of gravity significantly.

## Test Coverage Gaps

**All subsystems lack unit tests:**
- What's not tested: `SwerveDrive`, `SwerveModule`, `Elevator`, `PoseFilter`, `PoseEstimator`, `POIGenerator`
- Files: All `.cpp` files under `122-Swerve-Template/src/main/cpp/subsystems/` and `utils/`
- Risk: Kinematics bugs, PID value errors, sensor fusion logic errors go undetected until physical testing.
- Priority: High

**`PoseFilter` correctness is entirely untested:**
- What's not tested: The timestamp deduplication bug and queue-fill-before-accepting logic are not exercised in any test.
- Files: `122-Swerve-Template/src/main/include/utils/PoseFilter.h`
- Risk: Bad vision poses accepted during startup; robot teleports or misestimates position.
- Priority: High

**Elevator height fusion is untested:**
- What's not tested: `GetFusedHeight()`, `GetHallHeight()`, `AutoCalibrateHeight()`, `InterpolatePWL()`
- Files: `122-Swerve-Template/src/main/cpp/subsystems/Elevator.cpp` lines 287–535
- Risk: Height miscalculation leads to dangerous carriage movement outside physical bounds.
- Priority: High

---

*Concerns audit: 2026-06-22*
