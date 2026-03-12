# Changelog

## [Unreleased] - 2026-03-12

### Added
- **GPU fan control**: `ec_write_fan_duty` now simultaneously writes to both
  CPU fan (EC port `0x01`) and GPU fan (EC port `0x02`), keeping both fans
  always in sync at the same duty percentage
- GPU fan RPM registers `EC_REG_FAN2_RPMS_HI` (`0xD2`) and
  `EC_REG_FAN2_RPMS_LO` (`0xD3`) for reading GPU fan speed
- `fan2_rpms` field in shared memory for GPU fan RPM tracking
- `ec_query_fan2_rpms()` function for direct GPU fan RPM query
- Named constants `EC_CMD_FAN`, `EC_PORT_FAN_CPU`, `EC_PORT_FAN_GPU` to
  replace magic numbers `0x99`, `0x01`, `0x02`

### Changed
- `ui_update()` now uses the average RPM of CPU and GPU fans for the tray
  icon animation, instead of CPU fan only
- `main_dump_fan()` now displays CPU fan RPM and GPU fan RPM separately
- Adjusted `MAX_FAN_RPM` from `4400` to `5500` to match actual hardware
  (verified via EC register dump on target machine)

### Fixed
- **Auto mode silent failure**: `ec_write_fan_duty` had a minimum validation
  of 60%, but the auto duty algorithm could return 30/40/50%, causing write
  calls to silently fail and the fan to never decrease speed. Minimum is now
  30% to match the algorithm's lower bound.

---

## [Previous] - Ubuntu 22+ compatibility update

- Migrated from `appindicator3` to `ayatana-appindicator3` for Ubuntu 22+
- Updated build instructions

## [Initial]

- Original release: system tray indicator for Clevo laptop fan control
- Auto fan control algorithm based on CPU/GPU temperature
- Manual fan duty selection (60% ~ 100%) via tray menu
- Fork/shared-memory architecture: EC worker runs as root, UI runs as
  desktop user
