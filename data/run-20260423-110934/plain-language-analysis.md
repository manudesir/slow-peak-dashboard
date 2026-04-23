# Plain-Language Analysis For Run 20260423-110934

## What Happened

This 10-minute test captured the same kind of slow peak you described: the computer felt unusable even though normal resource counters were not maxed out.

The important detected times are:

| Time | What the monitor saw | Why it matters |
| --- | --- | --- |
| 11:12:08 | The monitor loop took 107 seconds. Process enumeration took 6.2 seconds and system performance counter queries took 9.9 seconds. | Windows itself was slow to answer basic system/process questions. This matches Task Manager or app switching becoming very slow. |
| 11:14:00 | The monitor expected to sample every 10 seconds, but the next sample arrived 112 seconds later. | The PowerShell monitor did not get scheduled normally for almost 2 minutes. This is a strong whole-system responsiveness signal. |
| 11:17:40 | System performance counter queries took 16.9 seconds. | Windows management/performance counters stalled again, likely matching the slow peak you noticed around 11:18. |

Your manual observations were added in `annotations.csv`:

| Time window | Manual note |
| --- | --- |
| 11:12 to 11:14 | You observed the machine was nearly unusable. |
| 11:18 to 11:20 | You observed another slow peak. |

## What It Was Not

The data does not look like a simple CPU/RAM/disk overload:

- CPU during detected spikes was about 20-25%, not maxed out.
- Memory commit was about 22-25%, far below pressure.
- Disk queue was 0 during the detected spikes.
- The SSD was not showing a sustained queue bottleneck in this run.

So the main symptom is not "Chrome used too much RAM" or "the disk was full." It looks more like Windows was intermittently unable to schedule or service basic UI/system work quickly.

## Most Suspicious Signals

The strongest signal is Windows management/performance counter latency:

- `ProcessQueryMs` became high at 11:12.
- `SystemQueryMs` became high at 11:12 and 11:17.
- The sample gap jumped to 112 seconds at 11:14.

Chrome also showed notable I/O around the 11:14 recovery point:

- Chrome reached about 21.5 MB/s I/O.
- Another Chrome process reached about 16.9 MB/s I/O.

That does not prove Chrome is the root cause. It means Chrome was active around the recovery period and may have contributed, triggered scanning, or simply resumed work after the stall.

## Current Hypotheses

Ranked from most likely to less likely based on this run and the earlier investigation:

1. Windows management/performance infrastructure is stalling.
   - Evidence: process and CIM/performance queries blocked for seconds to minutes.
   - This explains why Task Manager can take a long time to open.

2. A background driver/service/security layer is intermittently blocking UI or system calls.
   - Candidates supported by this run/context: Microsoft Defender/Endpoint, Intel/Lenovo management services, VPN/security components, and Logitech tooling.
   - This run alone does not identify the exact component.

3. Chrome activity triggers or amplifies the stall.
   - Evidence: Chrome had the largest process I/O near 11:14.
   - This could be cache/profile I/O, extension activity, network/VPN interaction, or Defender scanning Chrome files.

4. Windhawk is ruled out for this captured run.
   - Evidence: `processes.csv` contains no Windhawk process rows.
   - `services-on-spikes.csv` shows the Windhawk service was `Stopped` at 11:12, 11:14, and 11:17.
   - Therefore Windhawk should not be treated as a likely cause for this dataset.

## How To Inspect This In The Dashboard

Open `slow-peak-dashboard.html` and load the run folder:

`slow-peak-monitor-runs\run-20260423-110934`

Then:

1. In the timeline, keep these metrics enabled:
   - Sample gap ms
   - Scheduler delay ms
   - Monitor loop ms
   - System query ms
   - Process query ms
   - UI probe ms

2. Make sure these overlays are enabled:
   - Show annotations
   - Show detected spikes
   - Show computed spikes

3. Click the spike row at `11:12:08`.
   - Look at the Selected Time panel.
   - Then check Nearby Processes.
   - This shows what was active around the moment Windows process/system queries stalled.

4. Click the manual annotation at `11:18`.
   - This lets you compare what the monitor saw near the time you personally observed the issue.
   - In this run, the closest machine signal is the `11:17:40` system-query stall.

5. Use Process Analysis to sort by:
   - Max IO MB/s
   - Max CPU %
   - Max memory

6. Use the text filter for suspects:
   - `chrome`
   - `MsMpEng`
   - `MsSense`
   - `SSLVpnClient`
   - `IDBWM`
   - `WmiPrvSE`

## Who Is Responsible?

This run does not prove a single responsible process.

The responsible layer is more likely below normal app CPU usage: Windows management/performance APIs, a driver/service, security scanning, VPN/security software, or a vendor background service. Chrome is a visible participant around the stall, but the data does not prove Chrome alone is the root cause.

Windhawk is not supported as a cause for this run because it was not running.

Most likely suspects to test first:

1. Windows WMI/performance counter stack.
2. Chrome profile/extensions/cache activity.
3. Defender/Endpoint scanning or policy activity.
4. Securepoint VPN or network/security filtering.
5. Intel Dynamic Bandwidth Management / Lenovo platform services.
6. Logitech background tooling.

## How To Fix Or Narrow It Down

Recommended next tests:

1. Run another 10-minute monitor session with only Chrome open, as you did here.
2. Run a second 10-minute session in Chrome Incognito with extensions disabled.
3. Run a third 10-minute session with Securepoint VPN disconnected if possible.
4. If you use Securepoint VPN during the issue, run one test disconnected from VPN if possible.
5. Update or repair Intel/Lenovo drivers, especially Intel Connectivity Performance Suite / Dynamic Bandwidth Management.
6. If this is a managed work laptop, share this folder with IT and point them to:
   - `spikes.csv`
   - `metrics.csv`
   - `plain-language-analysis.md`

If the spikes disappear in Incognito/extensions-disabled Chrome, a Chrome extension or profile/cache interaction is likely.

If spikes continue with Chrome minimal and VPN disconnected, the issue is likely a lower-level driver, security, Windows management, or vendor service problem.
