# Codebase Concerns

**Analysis Date:** 2026-06-22

---

## CRITICAL: Deployment Target Mismatch

**The deploy configuration targets `SystemCore`, not a raw Raspberry Pi CM5.**

- Risk: The `build.gradle` deploy block uses `getTargetTypeClass('SystemCore')` and `useDefaultSystemcoreHostName()`. These GradleRIO helpers assume the target is a FIRST-managed SystemCore device running WPILib's custom OS image with the expected filesystem layout (`/home/systemcore/`, pre-installed HAL libraries, FRC robot startup service, etc.). A raw Raspberry Pi CM5 with a stock Debian/Bookworm image will not have any of this.
- Files: `build.gradle` lines 9–38
- Impact: `./gradlew deploy` will fail to connect or fail post-copy because the target user, paths, and startup hooks will not exist on a raw CM5.
- Fix approach: Either (a) flash the official WPILib SystemCore image onto the CM5 (recommended for alpha testing), or (b) manually configure the CM5 with the expected user accounts, library paths, and systemd service that GradleRIO's `WPILibNativeArtifact` artifact type expects. Option (b) requires reverse-engineering GradleRIO's deploy internals and is not documented.

---

## CRITICAL: HAL Compatibility on Raw Linux

**WPILib HAL is not hardware-agnostic — it requires SystemCore-specific drivers.**

- Risk: `wpi::StartRobot<Robot>()` in `src/main/cpp/Robot.cpp` (line 79) initializes the WPILib HAL. On a standard roboRIO or SystemCore, the HAL maps to hardware-specific kernel drivers (for CAN, PWM, DIO, SPI, I2C, analog I/O). On a raw CM5, those kernel modules and device nodes will not exist. The program will panic or segfault at `HAL_Initialize` (also called explicitly in `src/test/cpp/main.cpp` line 3).
- Files: `src/main/cpp/Robot.cpp` line 79, `src/test/cpp/main.cpp` line 3
- Impact: The robot program cannot start on bare Linux. Even if the binary transfers successfully, it will crash immediately at runtime.
- Fix approach: Use the WPILib simulation HAL (`wpi.platforms.desktop` target) for development/testing on a raw Linux host, or confirm that the SystemCore OS image includes the required HAL kernel modules before deploying to CM5.

---

## CRITICAL: No Actual Swerve Drive Code

**The project is a vanilla WPILib template — no swerve drive subsystem exists.**

- Risk: Despite being named "SwerveBaseTest", the codebase contains only the auto-generated WPILib command-based skeleton. There is no `SwerveSubsystem`, no swerve module abstraction, no kinematics, no odometry, and no motor/encoder references. The `ExampleSubsystem` does nothing (`ExampleCondition()` always returns `false`, `ExampleMethodCommand()` runs an empty lambda).
- Files: `src/main/cpp/subsystems/ExampleSubsystem.cpp`, `src/main/include/subsystems/ExampleSubsystem.hpp`
- Impact: The project cannot drive a robot. All actual swerve logic is presumably in the `122-Swerve-Template` git submodule, which is a separate codebase from a different team and is explicitly out of scope for this project.
- Fix approach: Integrate swerve drive code from the submodule or implement it directly in `src/main/cpp/subsystems/` and `src/main/include/subsystems/`.

---

## Alpha Software Stability Risks

**GradleRIO `2027.0.0-alpha-6` and WPILib `2027_alpha5` are pre-release software.**

- Risk: The `settings.gradle` pins to `2027_alpha5` local installation path and `build.gradle` requires GradleRIO `2027.0.0-alpha-6`. Alpha releases frequently have breaking API changes between drops. The `wpi/commands2/` namespace (e.g., `wpi::cmd::CommandScheduler`, `wpi::cmd::Trigger`) seen in source files reflects the alpha API surface, which may not be stable or match final 2027 release.
- Files: `settings.gradle` line 5, `build.gradle` line 4, all headers under `src/main/include/`
- Impact: Upgrading to a newer alpha or the stable release may require source changes. There is no pinned lockfile for WPILib itself beyond what GradleRIO resolves from the local `wpilibHome` maven directory.
- Fix approach: Track the WPILib SystemCore Testing GitHub discussions (referenced in `docs/wpilib-installation.md`) for breaking changes between alpha drops. Pin the GradleRIO plugin version explicitly in `build.gradle` (already done at alpha-6) and do not upgrade without testing.

---

## Missing Linux Runtime Configuration

**No `robot.json` or FRC service configuration exists for the target.**

- Risk: WPILib's robot startup mechanism on SystemCore relies on a `robot.json` configuration file and a systemd service managed by the WPILib deployment infrastructure. No such file exists in this project. The deploy artifact type `WPILibNativeArtifact` likely generates or expects this file on the target, but there is no local source for it.
- Files: Absent — no `robot.json` anywhere under `src/`
- Impact: Even if the binary is manually copied to the CM5, it will not auto-start on boot without the correct systemd unit configuration.
- Fix approach: If deploying via GradleRIO to a proper SystemCore image, the deploy plugin handles this. If deploying manually to raw CM5, create a systemd service unit that runs the compiled binary with appropriate permissions and restart policy.

---

## Tech Debt: Empty Deploy Directory

**`src/main/deploy/` contains only a placeholder text file.**

- Issue: `src/main/deploy/example.txt` is the only file in the deploy tree. The deploy block in `build.gradle` (line 30–35) copies this entire directory to `/home/systemcore/deploy` on the target. This is placeholder content from the WPILib project template and has no functional value.
- Files: `src/main/deploy/example.txt`, `build.gradle` lines 30–35
- Impact: No immediate runtime impact, but the deploy directory must be populated with any config files (e.g., path planner trajectories, calibration data) the robot program will read at runtime via `wpi::filesystem::GetDeployDirectory()`.
- Fix approach: Remove `example.txt` and add real deploy files as the robot program requires them, or set `deleteOldFiles = false` (already set) to avoid stale file cleanup issues.

---

## Tech Debt: Version Mismatch Between Preferences and GradleRIO

**`.wpilib/wpilib_preferences.json` declares `2027_alpha5` but GradleRIO plugin is alpha-6.**

- Issue: `.wpilib/wpilib_preferences.json` sets `projectYear` to `"2027_alpha5"`, while `build.gradle` uses GradleRIO version `2027.0.0-alpha-6` and `settings.gradle` also references `2027_alpha5` as the local installation folder name. The folder name `2027_alpha5` is a quirk of the installer (noted in `docs/wpilib-installation.md`), so this is intentional but confusing.
- Files: `.wpilib/wpilib_preferences.json` line 4, `build.gradle` line 4, `settings.gradle` line 5
- Impact: No build impact since the folder name is what matters for local maven resolution. However, it creates confusion when reading project metadata — the year string and the actual plugin version do not match.
- Fix approach: Document this discrepancy explicitly (as done in `docs/wpilib-installation.md`). Accept as a known quirk of the alpha installer.

---

## Test Coverage Gaps

**Only a HAL initialization stub exists — no actual tests are written.**

- What's not tested: All robot logic, command scheduling, subsystem behavior, and autonomous routines.
- Files: `src/test/cpp/main.cpp` (only test file — 10 lines, initializes HAL and runs GoogleTest with no registered test cases)
- Risk: Any logic added to `ExampleSubsystem`, `RobotContainer`, or future swerve subsystems will ship completely untested.
- Priority: Medium (acceptable for early template stage; becomes High once real robot logic is added)

---

## Dependencies at Risk: CommandsV2 vs CommandsV3 Conflict Guard

**`vendordeps/CommandsV2.json` hardcodes a conflict check against a CommandsV3 UUID.**

- Risk: The vendordep declares a `conflictsWith` entry for `CommandsV3.json` with UUID `4decdc05-a056-46cf-9561-39449bbb01306`. If CommandsV3 is ever added to the project (possible as the alpha library evolves), the build will fail with an error. The `mavenUrls` array is empty, meaning this vendordep resolves only from the local WPILib installation and not from any remote maven — the project cannot build without the local `2027_alpha5` WPILib installation present on the build machine.
- Files: `vendordeps/CommandsV2.json`
- Impact: The project is not portable to a CI environment without the full 2.7 GB WPILib installer being run first. There is no online fallback URL.
- Fix approach: Accept as a constraint of alpha WPILib development. Ensure all build machines run the WPILib installer before attempting a build. Consider adding a `mavenUrls` entry if WPILib publishes alpha artifacts to an online maven repository in future drops.

---

## Submodule Coupling Risk

**The `122-Swerve-Template` submodule is pinned to a non-stable branch.**

- Risk: `.gitmodules` pins the submodule to branch `DEV/brennan/system_core_update` — a development branch on an external team's repo. This branch can be force-pushed, rebased, or deleted without notice, which would break `git submodule update` for anyone cloning this repo.
- Files: `.gitmodules` lines 1–4
- Impact: Any developer doing a fresh clone and running `git submodule update --init --recursive` may get different code than the original developer, or may get an error if the branch is deleted.
- Fix approach: Pin the submodule to a specific commit SHA rather than a branch name. Update the SHA intentionally when pulling in upstream changes.

---

*Concerns audit: 2026-06-22*
