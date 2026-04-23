# Hardware Health Check - 2026-04-23

## Summary

There is credible evidence of a hardware, firmware, power, or low-level driver problem.

The strongest evidence is not the SSD, RAM usage, CPU usage, or GPU driver logs. It is the combination of:

- WHEA fatal hardware error events.
- Unexpected restart events at nearly the same times.
- A recorded bugcheck `0x0000019c`.
- Repeated firmware CPU speed-limiting warnings.

This does not prove which physical component is faulty. It does make a pure Chrome/software-only explanation unlikely.

## Evidence Found

### WHEA Hardware Errors

Windows logged `Microsoft-Windows-WHEA-Logger` event ID `1` errors:

- `2026-03-25 09:23:18`
- `2026-04-01 09:42:22`
- `2026-04-16 09:52:01`

These are fatal hardware error records. Windows stores most of the detail as raw WHEA binary data, so the exact component is not decoded from Event Viewer alone.

### Unexpected Restarts

Windows logged `Microsoft-Windows-Kernel-Power` event ID `41` critical events:

- `2026-03-25 09:23:03`
- `2026-04-01 09:42:10`
- `2026-04-16 09:51:48`

These line up closely with the WHEA errors. That makes the WHEA events important.

### Bugcheck

Windows logged `Microsoft-Windows-WER-SystemErrorReporting` event ID `1001`:

- `2026-04-01 09:42:20`
- bugcheck: `0x0000019c`
- parameters: `0x50, 0xffffc90eb8c44080, 0x0, 0x0`
- reported dump path: `C:\WINDOWS\Minidump\040126-14468-01.dmp`

The dump file was not present when checked, so it may have been cleaned up.

Microsoft documents bugcheck `0x19C` as `WIN32K_POWER_WATCHDOG_TIMEOUT`. It means Win32k did not turn the monitor on in time. Parameter `0x50` means the failure type was "calling monitor driver to power on".

Reference: https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-check-0x19c--win32k-power-watchdog-timeout

### CPU Speed Limited By Firmware

Windows logged many `Microsoft-Windows-Kernel-Processor-Power` event ID `37` warnings.

There were `300` warnings in the last 30 days, usually grouped as 20 events at a time, one per logical processor.

Recent groups:

- `2026-04-23 09:25`
- `2026-04-22 20:09`
- `2026-04-20 19:27`
- `2026-04-16 11:59`
- `2026-04-16 10:23`
- `2026-04-16 09:53`
- `2026-04-16 01:26`
- `2026-04-16 01:22`
- `2026-04-10 15:09`
- `2026-04-08 09:04`
- `2026-04-02 09:43`
- `2026-04-01 09:43`
- `2026-03-31 09:09`
- `2026-03-30 09:16`
- `2026-03-25 09:24`

The message says processor speed was limited by system firmware. This usually points to power, thermal, firmware, docking, charger, or platform-management behavior.

## Evidence Not Found

### SSD

The internal SSD reports:

- Model: `SKHynix_HFS512GEJ9X162N`
- Health: `Healthy`
- Operational status: `OK`
- DiskDrive status: `OK`

No disk/storage warning or error events were found in the last 30 days, except `volmgr` event ID `162`, which only says Windows generated a memory dump.

The storage reliability counters did not expose detailed temperature/wear/error values on this machine, so this is not a complete SMART report.

### GPU Driver

No Display, Intel graphics, NVIDIA, AMD, or DxgKrnl warning/error events were found in the last 30 days.

### Device Manager

No Device Manager problem devices were reported through `Win32_PnPEntity`.

### RAM

No Windows Memory Diagnostics result events were found in the last 365 days.

This means no recent built-in memory test result was found. It does not prove RAM is healthy.

### Temperature

Windows did not expose ACPI thermal zone temperature rows through `MSAcpi_ThermalZoneTemperature`.

The monitor was updated to collect physical metrics when Windows or hardware-monitor tools expose them. A new run is needed to capture those columns.

## Current Hypothesis

The most likely area is CPU/platform firmware/power/thermal/display-power handling.

The bugcheck points at monitor power-on timing. The repeated firmware CPU throttling points at platform management. The WHEA events and unexpected restarts make this more serious than normal browser slowness.

The SSD and GPU driver logs do not look like the main suspect from the evidence available.

## Recommended Next Tests

1. Run Lenovo Vantage or Lenovo Commercial Vantage hardware diagnostics.
   Include system board, CPU, memory, storage, fan, and thermal tests.

2. Update Lenovo BIOS/UEFI and Lenovo platform drivers.
   Prioritize BIOS, Intel graphics, chipset, Intel Management Engine, power management, and thermal drivers.

3. Test without a dock or external monitor.
   The bugcheck involved monitor power-on timing, so a dock, USB-C display path, or external monitor can matter.

4. Test with a different charger or power path.
   Firmware CPU-limiting events can be caused by power delivery or charger/dock negotiation.

5. Run one new 20 to 30 minute monitor capture with `-PublishToGitHubPages`.
   Keep the physical metrics enabled. Watch whether slow peaks line up with CPU throttling, sample gaps, or new WHEA/power events.

6. Run a real memory test.
   Use Windows Memory Diagnostic first, or MemTest86 overnight if the problem continues.

7. If WHEA, Kernel-Power 41, or bugcheck `0x19C` repeats after BIOS/driver/power-path tests, open a Lenovo support case.
   Include the WHEA timestamps, Kernel-Power timestamps, bugcheck code, and this report.
