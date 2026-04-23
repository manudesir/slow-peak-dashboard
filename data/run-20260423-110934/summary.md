# Slow Peak Monitor Summary

Generated: 2026-04-23T11:19:45.1839249+02:00

## Files

- Run folder: `C:\Users\EmmanuelDesir\Documents\Codex\2026-04-22-can-you-investigate-why-my-computer\slow-peak-monitor-runs\run-20260423-110934`
- Metrics: `metrics.csv`
- Processes: `processes.csv`
- Events: `events.csv`
- Spikes: `spikes.csv`
- Hardware health: `hardware-health.md`

## Extremes

| Metric | Time | Value |
| --- | --- | ---: |
| Sample gap seconds | 2026-04-23T11:14:00.4407368+02:00 | 112,09 |
| Monitor loop ms | 2026-04-23T11:12:08.3501022+02:00 | 107080 |
| UI probe max delay ms | 2026-04-23T11:14:00.4407368+02:00 | 81 |
| Windows input delay ms | 2026-04-23T11:15:40.5420755+02:00 |  |
| Disk queue | 2026-04-23T11:15:40.5420755+02:00 | 0 |

## Detected Spikes

| Time | Reasons | CPU % | Mem commit % | Disk queue | UI delay ms | Foreground |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| 2026-04-23T11:12:08.3501022+02:00 | MonitorLoop;ProcessEnumeration;CimPerfQuery | 21 | 22 | 0 | 12 |  |
| 2026-04-23T11:14:00.4407368+02:00 | SampleGap;SchedulerDelay | 25 | 22 | 0 | 81 |  |
| 2026-04-23T11:17:40.6473226+02:00 | CimPerfQuery | 20 | 25 | 0 | 21 |  |

## Notes

- A high sample gap or UI probe delay with low CPU and memory points to OS/UI responsiveness stalls.
- A high process or CIM query duration means Windows management/process enumeration stalled.
- Foreground responsiveness uses a short WM_NULL timeout and does not send keystrokes or mouse input.

## Plain-Language Explanation

This run captured slow peaks that match the user report. At `11:12:08`, the monitor loop took `107080 ms`, process enumeration took `6177 ms`, and system/performance counter queries took `9892 ms`. At `11:14:00`, the monitor expected a 10-second interval but the next sample arrived `112.09 seconds` later. At `11:17:40`, system/performance counter queries took `16911 ms`.

This does **not** look like a simple CPU, memory, or disk overload. CPU was about `20-25%`, memory commit was about `22-25%`, and disk queue stayed at `0` during the detected spikes.

The strongest hypothesis is that Windows management/performance infrastructure, or something underneath it such as a driver, security layer, VPN filter, or vendor service, is intermittently stalling. Chrome had high I/O near the 11:14 recovery point, so Chrome may be involved, but this run does not prove Chrome is the root cause.

Windhawk is ruled out for this captured run: `processes.csv` contains no Windhawk process rows, and `services-on-spikes.csv` shows the Windhawk service was `Stopped` at all detected spike times.

Manual observations were added in `annotations.csv` for `11:12-11:14` and `11:18-11:20`. For a full explanation and next tests, see `plain-language-analysis.md`.
