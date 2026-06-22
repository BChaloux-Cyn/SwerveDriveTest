# Testing Patterns

**Analysis Date:** 2026-06-22

## Test Framework

**Runner:**
- Google Test (gtest), integrated via WPILib's GradleRIO Gradle plugin
- Config: `build.gradle` — `testSuites { wpilibUserProgramTest(GoogleTestTestSuiteSpec) }`
- Test sources: `src/test/cpp/`

**Assertion Library:**
- Google Test (`gtest/gtest.h`)

**HAL Initialization:**
- `HAL_Initialize(500, 0)` MUST be called before `RUN_ALL_TESTS()` — required for WPILib simulation layer
- Entry point: `src/test/cpp/main.cpp`

**Run Commands:**
```bash
./gradlew test                  # Run all tests (Windows: gradlew.bat test)
./gradlew runWpilibUserProgramTestWindowsx86-64DebugGoogleTestExe   # Run debug test exe directly
```

## Test File Organization

**Location:** `src/test/cpp/` — separate from main sources in `src/main/cpp/`

**Naming:** Test files placed in `src/test/cpp/` with `.cpp` extension; no `.hpp` test headers currently present

**Current structure:**
```
src/test/cpp/
└── main.cpp          # Test runner entry point (HAL init + RUN_ALL_TESTS)
```

**No test cases are currently written** — only the test runner harness exists. The project is scaffolded for testing but has zero `TEST()` or `TEST_F()` blocks.

## Test Entry Point

`src/test/cpp/main.cpp`:
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

- Always preserve `HAL_Initialize(500, 0)` before `InitGoogleTest` — WPILib simulation crashes without it
- Do NOT add a second `main()` — this file is the single entry point for all tests

## Robot Code / Test Separation

The main robot binary guards its `main()` with:
```cpp
#ifndef RUNNING_WPILIB_TESTS
int main() {
  return wpi::StartRobot<Robot>();
}
#endif
```
(`src/main/cpp/Robot.cpp` line 77-81)

The `GoogleTestTestSuiteSpec` in `build.gradle` defines `RUNNING_WPILIB_TESTS` automatically when compiling the test suite, preventing duplicate `main()` symbols.

## Build Configuration

From `build.gradle`:
```groovy
testSuites {
    wpilibUserProgramTest(GoogleTestTestSuiteSpec) {
        testing $.components.wpilibUserProgram   // links against main program sources
        sources.cpp {
            source {
                srcDir 'src/test/cpp'
                include '**/*.cpp'
            }
        }
        wpi.cpp.vendor.cpp(it)
        wpi.cpp.deps.wpilib(it)
        wpi.cpp.deps.googleTest(it)
    }
}
```

Key: `testing $.components.wpilibUserProgram` compiles all main `.cpp` sources into the test binary — no need to re-include them in test files.

## What is Testable (WPILib Simulation Patterns)

**Subsystem logic:**
- `ExampleSubsystem::ExampleCondition()` returns a boolean — directly unit-testable
- `Periodic()` / `SimulationPeriodic()` can be called directly in tests after HAL init
- Hardware abstractions (motors, sensors) should be mocked via WPILib HAL simulation layers for hardware-dependent tests

**Command behavior:**
- Commands can be tested by instantiating them, calling `Initialize()`, `Execute()`, `IsFinished()` in sequence
- `wpi::cmd::CommandScheduler` is a singleton — reset between tests with `CommandScheduler::GetInstance().CancelAll()` or by re-initializing

**Auto routines:**
- `autos::ExampleAuto(subsystem*)` returns a `CommandPtr` — can be scheduled and stepped through in simulation

**What requires HAL simulation:**
- Any code touching hardware (motors, sensors, encoders) requires HAL sim
- `HAL_Initialize(500, 0)` in `main.cpp` enables this for all tests in the suite

**What is NOT testable directly:**
- `wpi::StartRobot<Robot>()` — the full robot loop (runs indefinitely)
- Actual hardware I/O without WPILib HAL simulation or mock layers

## Writing New Tests

Add `.cpp` files to `src/test/cpp/`. They are automatically included by the `**/*.cpp` glob in `build.gradle`.

Pattern for a new test file:
```cpp
#include "gtest/gtest.h"
#include "subsystems/ExampleSubsystem.hpp"  // project headers via double-quotes

TEST(ExampleSubsystemTest, ConditionDefaultsFalse) {
    ExampleSubsystem subsystem;
    EXPECT_FALSE(subsystem.ExampleCondition());
}
```

Pattern for fixture-based tests:
```cpp
#include "gtest/gtest.h"
#include "subsystems/ExampleSubsystem.hpp"

class ExampleSubsystemTest : public ::testing::Test {
 protected:
  ExampleSubsystem subsystem;
};

TEST_F(ExampleSubsystemTest, ConditionDefaultsFalse) {
  EXPECT_FALSE(subsystem.ExampleCondition());
}
```

## Coverage

**Requirements:** None enforced — no coverage tooling configured.

**Current state:** 0 test cases written. Framework fully scaffolded and operational.

## Test Types

**Unit Tests:**
- Pure logic in subsystem methods (`ExampleCondition`, command `IsFinished`, etc.)
- No hardware required, no HAL sim calls needed beyond initialization

**Simulation Tests:**
- Tests that invoke periodic methods or hardware abstractions
- Require `HAL_Initialize` (already present in `main.cpp`)
- Use WPILib's HAL simulation layer to stub hardware state

**E2E / Integration Tests:**
- Not configured. WPILib does not provide a built-in end-to-end framework; simulation-based integration tests would be added to `src/test/cpp/` alongside unit tests.

---

*Testing analysis: 2026-06-22*
