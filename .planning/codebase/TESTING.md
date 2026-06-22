# Testing Patterns

**Analysis Date:** 2026-06-22

## Test Framework

**Runner:**
- GoogleTest (GTest) — linked via WPILib's Gradle dependency helper.
- Root project config: `build.gradle` test suite `wpilibUserProgramTest`, plugin `google-test-test-suite`.
- Submodule config: `122-Swerve-Template/build.gradle` test suite `frcUserProgramTest`.
- Both projects use the same GTest runner pattern.

**Assertion Library:**
- GoogleTest built-in macros (`EXPECT_*`, `ASSERT_*`, `GTEST_*`).

**HAL Initialization:**
- WPILib requires HAL initialization before any hardware-interacting code runs in tests.
- Both projects initialize HAL in `main()` before running tests.

**Run Commands:**
```bash
./gradlew test                  # Run all tests (root project)
./gradlew wpilibUserProgramTest # Run tests (root project, explicit target)

cd 122-Swerve-Template
./gradlew test                  # Run all tests (submodule)
./gradlew frcUserProgramTest    # Run tests (submodule, explicit target)
```

## Test File Organization

**Location:**
- Tests live in a separate directory tree mirroring the main source layout.
- Root project: `src/test/cpp/`
- Submodule: `122-Swerve-Template/src/test/cpp/`

**Naming:**
- Test entry point: `main.cpp` (required GTest harness file).
- Additional test files would follow the pattern `*Test.cpp` or `*_test.cpp` (no additional test files currently exist beyond the harness).

**Structure:**
```
src/
  main/
    cpp/          # Production code
    include/      # Production headers
  test/
    cpp/          # Test code
      main.cpp    # GTest + HAL entry point
```

## Test Entry Point (main.cpp)

Both projects share the identical test harness pattern:

**Root project** (`src/test/cpp/main.cpp`):
```cpp
#include <wpi/hal/HAL.h>
#include "gtest/gtest.h"

int main(int argc, char** argv) {
  HAL_Initialize(500, 0);
  ::testing::InitGoogleTest(&argc, argv);
  int ret = RUN_ALL_TESTS();
  return ret;
}
```

**Submodule** (`122-Swerve-Template/src/test/cpp/main.cpp`):
```cpp
#include <hal/HAL.h>
#include "gtest/gtest.h"

int main(int argc, char** argv) {
  HAL_Initialize(500, 0);
  ::testing::InitGoogleTest(&argc, argv);
  int ret = RUN_ALL_TESTS();
  return ret;
}
```

Note: The HAL header path differs — `<wpi/hal/HAL.h>` (root, newer API) vs `<hal/HAL.h>` (submodule, stable API). Use the header that matches the project's GradleRIO version.

`HAL_Initialize(500, 0)` must always precede `InitGoogleTest`. The `500` argument sets the HAL timeout in milliseconds.

## Current Test Coverage

**Actual test cases:** None. Both projects contain only the GTest entry point (`main.cpp`) and no test files with `TEST()` or `TEST_F()` definitions. The test infrastructure is wired up and builds successfully, but no tests are written.

**Build verification:**
- The Gradle build compiles both production code and test code as separate executables.
- `wpi.cpp.deps.googleTest(it)` links GTest into the test binary.
- `testing $.components.wpilibUserProgram` (root) / `testing $.components.frcUserProgram` (submodule) means the test executable is built from the same production sources plus test sources.

## How to Add Tests

Add `.cpp` files to `src/test/cpp/`. They are automatically included via the Gradle glob `include '**/*.cpp'`. No registration or CMakeLists update required.

**Minimal test file example:**
```cpp
#include "gtest/gtest.h"

// Test a pure computation function that does not touch hardware
TEST(MathUtilNK, DeadbandZeroWhenBelowThreshold) {
    double result = MathUtilNK::calculateAxis(0.05, 0.15);
    EXPECT_DOUBLE_EQ(result, 0.0);
}

TEST(MathUtilNK, DeadbandScalesAboveThreshold) {
    double result = MathUtilNK::calculateAxis(0.5, 0.15);
    EXPECT_GT(result, 0.0);
    EXPECT_LT(result, 1.0);
}
```

Include the header for the code under test:
```cpp
#include "Constants.hpp"  // for MathUtilNK::calculateAxis
```

## Mocking

**Framework:** No mocking framework is currently configured (no GMock usage found, though GMock ships with GTest and is available).

**Hardware interaction in tests:**
- HAL simulation mode is enabled by `HAL_Initialize(500, 0)` — this runs the HAL in simulation mode, allowing motor controller and sensor objects to instantiate without real hardware.
- Classes that directly construct hardware objects (e.g., `SwerveDrive`, `SwerveModule`) are difficult to unit test directly due to CAN bus initialization requirements.
- Testable code: pure functions, math utilities, state machine logic, data transformation functions.

**What to test without mocks:**
- `MathUtilNK::calculateAxis()` in `122-Swerve-Template/src/main/include/Constants.hpp` — pure function, no hardware dependency.
- Waypoint/path calibration logic in `122-Swerve-Template/PathCalibrator/` Python scripts.
- Any new utility/math classes that do not hold hardware members.

**What requires simulation or mocks:**
- Subsystem classes (`SwerveDrive`, `SwerveModule`, `Elevator`, etc.) — instantiate hardware.
- `RobotContainer` / `Robot` — require full HAL simulation context.

## Fixtures and Factories

**Test Data:** Not established. No fixture files or factory helpers exist.

**Recommended pattern when adding tests:**
```cpp
class MathTest : public ::testing::Test {
 protected:
  double deadband = 0.15;
};

TEST_F(MathTest, BelowDeadbandReturnsZero) {
    EXPECT_EQ(MathUtilNK::calculateAxis(0.1, deadband), 0.0);
}
```

## Coverage

**Requirements:** None enforced. No coverage tooling configured in `build.gradle`.

**View Coverage:**
- Not currently available. Would require adding `--coverage` GCC flags and a coverage report plugin to Gradle.

## Test Types

**Unit Tests:**
- Target: Pure computational logic with no hardware or WPILib scheduler dependencies.
- Candidates: `MathUtilNK::calculateAxis`, path geometry math, POI distance calculations.

**Integration Tests (HAL Simulation):**
- HAL is initialized in the test harness, enabling simulation-mode instantiation of WPILib hardware objects.
- Subsystem integration tests could verify scheduler interactions using `wpi::cmd::CommandScheduler::GetInstance()`.

**E2E Tests:**
- Not used. Robot E2E testing is performed by physically deploying to the robot or running WPILib's desktop simulation GUI (launched via `./gradlew simulateNative`).

## Python Tests

No Python test framework (e.g., `pytest`, `unittest`) is configured for the `PathCalibrator/` scripts. The scripts are standalone utilities; tests would be added with a `pytest` setup if needed.

## Simulation Build

The build system also supports a simulation target (separate from tests):
```bash
./gradlew simulateNative   # Launch WPILib simulation GUI
```

This is the primary integration verification method for robot code, not automated tests.

---

*Testing analysis: 2026-06-22*
