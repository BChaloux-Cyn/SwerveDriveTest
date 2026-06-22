# WPILib 2027 Installation

## Release Info

- **Release page:** https://github.com/wpilibsuite/allwpilib/releases/tag/v2027.0.0-alpha-6
- **Docs:** https://docs.wpilib.org/en/2027/

## Downloads

| Platform | Link | Size |
|---|---|---|
| Windows | https://packages.wpilib.workers.dev/installer/v2027.0.0-alpha-6/Win64/WPILib_Windows-2027.0.0-alpha-6.iso | 2.6 GB |
| macOS (Arm) | https://packages.wpilib.workers.dev/installer/v2027.0.0-alpha-6/macOSArm/WPILib_macOS-Arm64-2027.0.0-alpha-6.dmg | 2.2 GB |
| macOS (Intel) | https://packages.wpilib.workers.dev/installer/v2027.0.0-alpha-6/macOS/WPILib_macOS-Intel-2027.0.0-alpha-6.dmg | 2.3 GB |
| Linux (x64) | https://packages.wpilib.workers.dev/installer/v2027.0.0-alpha-6/Linux/WPILib_Linux-2027.0.0-alpha-6.tar.gz | 2.7 GB |

## System Requirements

- 64-bit Windows 11, Ubuntu 26.04, or macOS 15 or higher
- C++ developers: latest Visual Studio
- macOS: Xcode Command Line Tools

## Windows Installation Steps

1. Download the `.iso` file from the link above
2. Double-click the ISO in File Explorer — Windows mounts it as a virtual drive automatically
3. Open the virtual drive and run `WPILibInstaller.exe`
4. Select **"Everything"** when prompted for install options
5. Wait for installation to complete (installs to `C:\Users\Public\wpilib\2027_alpha5\`)
6. Right-click the virtual drive in File Explorer and select **Eject** to unmount the ISO

## Notes

- Despite downloading alpha-6, the installer places files under the folder name `2027_alpha5` — this is expected
- GradleRIO version available: `2027.0.0-alpha-6`
- Includes the SystemCore cross-compilation toolchain targeting `aarch64-bookworm-linux-gnu`
- SystemCore alpha testing discussions: https://github.com/wpilibsuite/SystemcoreTesting/
