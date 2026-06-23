# WPILib CM5 Robot Platform

## What This Is

An experimental FRC robot control platform running WPILib 2027 alpha5 on a raw Raspberry Pi Compute Module 5 (CM5) with IO board — not a pre-configured SystemCore or roboRIO. A small team (2-4 people) is exploring whether a bare CM5 can serve as a viable FRC robot controller, starting with getting the Driver Station to connect and robot modes to work over Ethernet.

## Core Value

The Driver Station connects to the CM5 over Ethernet and robot mode switching (enable/disable) works reliably.

## Requirements

### Validated

- ✓ WPILib 2027 alpha5 C++ project structure initialized — existing
- ✓ Gradle build system configured for WPILib 2027 toolchain — existing
- ✓ Command-Based robot program skeleton (Robot, RobotContainer, subsystem/command structure) — existing

### Active

- [ ] CM5 has a suitable Linux OS flashed and boots reliably
- [ ] CM5 has a static IP configured for FRC Driver Station communication
- [ ] WPILib aarch64 HAL and robot binary successfully cross-compiled from this project
- [ ] Robot binary deploys to CM5 (via Gradle deploy task or equivalent)
- [ ] Robot program launches automatically on CM5 boot (systemd service)
- [ ] FRC Driver Station detects the robot over Ethernet
- [ ] Robot mode switching works: Disabled → Teleop Enabled → Disabled

### Out of Scope

- Hardware I/O (CAN bus, PWM, motors, sensors) — Milestone 1 is connectivity only; hardware drivers are a separate research effort
- Swerve drive implementation — depends on hardware I/O being proven first
- WiFi connectivity — Ethernet tethered connection only for now; WiFi adds network config complexity
- PathPlanner / autonomous routines — no motion capability yet
- Full FRC field compliance — not targeting competition hardware in this milestone

## Context

- The CM5 is a raw compute module (not a FIRST-supplied SystemCore). It requires a carrier/IO board to expose USB, Ethernet, and GPIO.
- WPILib 2027 is alpha software specifically being developed toward CM5-class hardware; the SystemCore is the official FIRST product built on the CM5, but this project uses a bare module.
- The key open question is HAL compatibility: WPILib's HAL normally targets specific hardware (roboRIO, SystemCore). On a raw CM5, GPIO/CAN bindings may not resolve, but the robot program may still boot and communicate with the DS in a "simulation-like" mode where robot logic runs but hardware I/O is stubbed.
- OS choice (Raspberry Pi OS Lite 64-bit vs. Ubuntu Server) is a decision to be made during Phase 1 research.
- Cross-compilation target is `aarch64-linux-gnu` — the existing Gradle build targets roboRIO (ARM32 + NI RTOS), so the toolchain configuration will need to change.

## Constraints

- **Hardware**: Raw CM5 + IO board only — no roboRIO, no SystemCore, no FRC CAN hardware initially
- **Software**: WPILib 2027 alpha5 — pre-release, docs/APIs may change; pin to specific alpha tag
- **Network**: Ethernet only (direct or via switch) — FRC DS talks UDP to robot on standard FRC IP scheme
- **Team size**: 2-4 people, student/mentor mix — solutions must be documentable for the whole team
- **Alpha risk**: WPILib 2027 HAL for Linux/CM5 may have gaps; fallback is running in sim mode

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Ethernet over WiFi for first milestone | Fewer variables; isolate connectivity from network config complexity | — Pending |
| DS-connects-and-modes-work as milestone scope | Hardware I/O is a separate research problem; prove the control path first | — Pending |
| Raw CM5 instead of official SystemCore | Exploring feasibility with available hardware; SystemCore path would be straightforward | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-06-22 after initialization*
