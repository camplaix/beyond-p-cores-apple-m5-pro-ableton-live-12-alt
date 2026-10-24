# Ableton Live 12
# Beyond P-Cores, Part 2: How Apple's 15-Core M5 Pro Scales in Ableton Live 12 — The 10-Core Audio Ceiling, Super Core Contention, and Multi-Track Buffer Inversion

*Follow-up to [Part 1: How Apple's 3-Tier Silicon (M5/M6) Impacts Low-Latency DSP, Buffer Scaling, and Core Allocation](https://camplaix.github.io/beyond-p-cores-apple-m6-ableton-live-12/).*

In [Part 1](https://camplaix.github.io/beyond-p-cores-apple-m6-ableton-live-12/), testing the base 12-core Mac mini M6 exposed the compromises of a 2-Super + 4-Performance + 6-Efficiency layout in Ableton Live 12. With only two top-tier Super Cores, the M6 spilled onto its slower mid-tier P-cores as soon as a second track was added, and raising the buffer to 64 samples on a single third-party plugin chain demoted the audio thread off the Super Cores entirely.

That left an obvious follow-up question: what happens on the **Mac mini M5 Pro**, where Apple drops Efficiency cores altogether and gives you **5 Super Cores alongside 10 mid-tier Performance cores** (15 cores in total)?

With five Super Cores instead of two, and ten P-cores behind them, the M5 Pro looks on paper like it should solve the M6's early spillover bottleneck and scale smoothly up to 15 parallel tracks. Testing the 15-core M5 Pro across 1 to 16 channels with the exact same methodology tells a much more interesting story:

1. **Tracks 1 through 5 stay on the 5 Super Cores at 32 samples**—fixing the M6's Track 2 P-core spill—yet Waves L4 still loses nearly 30% of its per-track capacity across those five Super Cores while native Ableton Echo loses less than 7%.
2. **Real-time audio never uses more than 10 of the 15 cores.** Across every multi-track test from 9 to 16 channels, one 5-core P-group always sits out of audio processing while handling low-level background/system activity, triggering a steep 2:1 track-sharing cliff at **Track 11**.
3. **Buffer inversion isn't limited to a single track.** At 128 samples, even a 5-track Waves L4 session has one of its five tracks pulled onto a P-core while a Super Core sits half-empty, bottlenecking the entire project.

## Test Rig & Methodology

- **Apple Silicon (New):** Mac mini M5 Pro (15-Core: 5 Super Cores, 10 mid-tier Performance Cores, 0 Efficiency Cores; 24 GB Unified Memory) running macOS Golden Gate. Note that Activity Monitor numbers the M5 Pro inversely compared to the M6: **Cores 1–10 are Performance cores**, and **Cores 11–15 are Super Cores**. Across all captures, load patterns divide cleanly into three 5-core groups: **Cores 1–5 (P-group A)**, **Cores 6–10 (P-group B)**, and **Cores 11–15 (Super Cores)**.
- **Apple Silicon (Reference from Part 1):** Mac mini M6 (12-Core: Cores 1–6 Efficiency, Cores 7–8 Super, Cores 9–12 Performance; 24 GB) on macOS Golden Gate.
- **x86 Reference (from Part 1):** Intel Core Ultra 7 270K Plus (8 Lion Cove P-cores used for MMCSS audio threads, locked at $5.4\text{ GHz}$, C-States off, DDR5 CUDIMM $8800\text{ MT/s}$) on Windows 11 Pro 25H2.
- **Audio Interfaces & Drivers:** RME Babyface Pro FS (USB 2.0 @ $44.1\text{ kHz}$, 32 samples on macOS via RME's latest DriverKit driver; 48 samples minimum on Windows ASIO), plus an internal RME HDSPe AIO Pro (PCIe @ $44.1\text{ kHz}$, 32 samples) on the Intel rig.
- **Methodology:** Every track in a session carries an identical plugin chain—either native **Ableton Echo** or third-party **Waves L4 Ultramaximizer**. Instances are raised or lowered equally across all tracks until the highest dropout-free count is found. *(Note: where a screenshot window title shows 1–4 fewer instances than the tables—such as `09ch Echo 73` vs. `77` final, or `15ch Echo 42` vs. `43` final—the screenshot was captured just before raising the session to its final dropout limit.)*

---

## Single-Track Headroom & Tracks 1–5: Staying on the Super Cores

On a single track at 32 samples, the 15-core M5 Pro reaches **90 instances of Ableton Echo** and **44 instances of Waves L4** before dropouts—trailing the newer M6 Super Core (102 Echo, 52 L4) by $-11.8\%$ and $-15.4\%$, respectively, and landing essentially level with the Intel 270K Plus on PCIe for Echo (89 instances).

Looking at Activity Monitor during a 1-track test shows a subtle difference in how the two plugins sit on the five Super Cores (Cores 11–15):

![1 track Echo vs Waves L4 at 32 samples](images/01ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2090.png)\
<sub>*1 track, 32 samples, Ableton Echo (90 instances): load sits primarily on Super Core 11 (~80%) with secondary spillover onto Super Core 12 (~25%), while P-cores 1–10 carry no heavy load.*</sub>

![1 track Waves L4 at 32 samples](images/01ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2044.png)\
<sub>*1 track, 32 samples, Waves L4 (44 instances): the single audio thread rotates across all five Super Cores (Cores 11–15), while P-cores 1–5 show light background/kernel activity and P-cores 6–10 sit idle.*</sub>

With Ableton Echo, the single track stays mostly on Core 11 with a small share on Core 12. With Waves L4, the thread hops across all five Super Cores (11 through 15).

What happens when we add tracks 2 through 5? On the M6 in Part 1, Track 2 immediately spilled onto a mid-tier P-core because the chip only had two Super Cores. On the M5 Pro, the numeric drop on Waves L4 from Track 1 to Track 2 (**44 down to 37 per track**, $-15.9\%$) initially looked like the exact same P-core spillover.

The CPU History captures prove otherwise:

![2 tracks Waves L4 at 32 samples](images/02ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2037.png)\
<sub>*2 tracks, 32 samples, Waves L4 (37/track = 74 total): both tracks remain on Super Cores 11–15. P-cores 6–10 sit at zero, and P-cores 1–5 show only light background/kernel activity.*</sub>

At 2 tracks of Waves L4, **there is no P-core spill**. Both tracks run on Super Cores 11–15 while P-cores 1–10 carry no heavy audio load. And that remains true all the way to **5 tracks** for both plugins:

![5 tracks Echo at 32 samples](images/05ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2084.png)\
<sub>*5 tracks, 32 samples, Ableton Echo (84/track = 420 total): all five Super Cores (11–15) are loaded 1:1.*</sub>

![5 tracks Waves L4 at 32 samples](images/05ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2031.png)\
<sub>*5 tracks, 32 samples, Waves L4 (31/track = 155 total): just like Echo, all five tracks run on Super Cores 11–15 with no heavy load on P-cores 1–10.*</sub>

Even though both plugins stay on the five Super Cores from 1 to 5 tracks (captured at 1, 2, and 5 tracks, with 3 and 4 interpolated), their per-track scaling within the Super Core group is completely different:

| Tracks (32 samples) | Active Audio Cores | Ableton Echo / Track (Total) | Echo vs. 1 Track | Waves L4 / Track (Total) | Waves L4 vs. 1 Track |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **1** | 1 Super Core (11–15) | **90** (90) | $100\%$ | **44** (44) | $100\%$ |
| **2** | 2 Super Cores (11–15) | **87** (174) | $-3.3\%$ | **37** (74) | $-15.9\%$ |
| **3** | 3 Super Cores *(inferred)* | **86** (258) | $-4.4\%$ | **35** (105) | $-20.5\%$ |
| **4** | 4 Super Cores *(inferred)* | **86** (344) | $-4.4\%$ | **32** (128) | $-27.3\%$ |
| **5** | **5 Super Cores (11–15)** | **84** (420) | **$-6.7\%$** | **31** (155) | **$-29.5\%$** |

With native **Ableton Echo**, five Super Cores behave almost like five independent processors: per-track headroom barely budges from **90 down to 84** ($-6.7\%$), allowing the M5 Pro to edge out the M6 at 3, 4, and 5 tracks (420 vs. 415 Echo instances at 5 tracks) despite the M6's faster individual Super Cores.

With **Waves L4**, per-track capacity drops $-15.9\%$ at 2 tracks (with three Super Cores still idle) and **$-29.5\%$ by 5 tracks** (44 down to 31 per track). Because Echo loses only $6.7\%$ across the exact same five Super Cores, a general multi-core clock drop doesn't explain it. The penalty is specific to Waves L4—whether from shared L2 cache/memory bandwidth contention across the Super Core group, plugin-internal shared state, or the overhead of migrating threads across all five Super Cores.

---

## Tracks 6–10: P-Core Spillover and the 10-Core Audio Ceiling

Adding a **6th track** pushes the session past the five Super Cores and forces the first track onto a mid-tier P-core.

From **Track 6 through Track 8**, per-track headroom locks into a flat plateau on both plugins:
- **Ableton Echo:** Drops from 84/track (5ch) to **81/track at 6 channels** (486 total) and holds steady at **81/track at 8 channels** (648 total).
- **Waves L4:** Drops from 31/track (5ch) to **29/track at 6 channels** (174 total), **29/track at 7 channels** (203 total), and **29/track at 8 channels** (232 total). Because the M5 Pro's P-cores top out at 29 L4 instances here versus 35 on the M6, the 12-core M6 actually beats the 15-core M5 Pro by **$+20.7\%$ at 6 tracks of Waves L4** (210 vs. 174).

At **9 and 10 tracks**, per-track numbers step down slightly—to **77 then 75 on Echo**, and **27 then 26 on Waves L4**—as the active 5-core P-group fills up:

![9 tracks Echo at 32 samples](images/09ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2073.png)\
<sub>*9 tracks, 32 samples, Ableton Echo (captured at 73/track; final CSV limit 77/track = 693 total): 9 cores carry heavy audio load (P-cores 6–9 and Super Cores 11–15). Core 10 sits idle, while P-cores 1–5 handle light background/system tasks.*</sub>

![10 tracks Echo at 32 samples](images/10ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2075.png)\
<sub>*10 tracks, 32 samples, Ableton Echo (75/track = 750 total, session peak): all 5 Super Cores (11–15) and all 5 P-cores in group B (6–10) are saturated 1:1. P-cores 1–5 carry only background/system load.*</sub>

![10 tracks Waves L4 at 32 samples](images/10ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2026.png)\
<sub>*10 tracks, 32 samples, Waves L4 (26/track = 260 total, session peak): P-cores 6–10 are flat-topped near 100%, while Super Cores 11–15 run around ~75–80% bar height because the slower P-cores set the dropout limit for the uniform session.*</sub>

Look closely at the 10-track Waves L4 capture above: **P-cores 6–10 are flat-topped at the ceiling, while Super Cores 11–15 have roughly 20–25% visible headroom left unused** because the slowest core in a uniform session dictates the dropout point. Even so, having 10 active cores at 1:1 saturation gives the M5 Pro its highest total throughput right here at **10 tracks**: **750 Echo instances** and **260 Waves L4 instances** (raising the buffer to 128 samples at 10 tracks gives Echo 74/track = 740, no gain over 32 samples).

---

## The Track 11 Cliff: Why 5 of the 15 Cores Never Run Audio

On a 15-core processor with 5 Super Cores and 10 P-cores, adding Track 11 should simply recruit a 6th P-core from Cores 1–5.

It doesn't. Across every multi-track test from 9 to 16 channels, **Ableton Live never puts heavy audio load on more than 10 cores simultaneously**:

![11 tracks Echo at 32 samples](images/11ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2045.png)\
<sub>*11 tracks, 32 samples, Ableton Echo (45/track = 495 total): adding Track 11 does not wake up Cores 1–5. All 11 tracks are forced to share the same 10 cores (Cores 6–15), dropping per-track capacity by 40%.*</sub>

![11 tracks Waves L4 at 32 samples](images/11ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2019.png)\
<sub>*11 tracks, 32 samples, Waves L4 (19/track = 209 total): the same 10-core ceiling holds. Cores 1–5 remain on light background duty while 11 tracks share Cores 6–15.*</sub>

Because Track 11 does not spill onto Cores 1–5, **one of the 10 active cores now has to process two tracks in series within the same 32-sample callback window** (2:1 track sharing). Since every track carries the same plugin count, the single core running two tracks hits the dropout wall at roughly half the per-track count, dragging the entire session down with it:
- **Ableton Echo (10 → 11 tracks):** Drops from **75/track down to 45/track ($-40.0\%$)**, slashing total project capacity from **750 down to 495 instances** ($-34.0\%$). Raising the buffer to 128 samples shows the exact same cliff: 10 tracks at 128 samples sustains **74/track (740)**, while 11 tracks drops to **47/track (517)**.
- **Waves L4 (10 → 11 tracks):** Drops from **26/track down to 19/track ($-26.9\%$)**, cutting total project capacity from **260 down to 209 instances** ($-19.6\%$).

### Interchangeable P-Core Groups (Cores 1–5 vs. Cores 6–10)

Is there something special about Cores 1–5 that prevents them from running audio? No—as the 12-, 15-, and 16-track captures show, the two 5-core P-groups can trade places:

![12 tracks Echo at 32 samples](images/12ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2045.png)\
<sub>*12 tracks, 32 samples, Ableton Echo (45/track = 540 total): here, heavy audio load runs on P-cores 1–5 plus Super Cores 11–15, while P-cores 6–10 take over the low-level background (red) kernel activity.*</sub>

In most multi-track captures, heavy audio runs on **Cores 6–10** while **Cores 1–5** show low-level background and red kernel bars. In three captures—`12ch Echo 45`, `15ch WavesL4 17`, and `16ch WavesL4 14`—that assignment flips: **Cores 1–5** run the heavy audio threads while **Cores 6–10** handle the background/kernel work. The two P-groups are completely interchangeable in performance (`11ch Echo` on Cores 6–10 and `12ch Echo` on Cores 1–5 both top out at 45 instances/track), and the background (red) load always sits in whichever P-group isn't carrying audio.

Whether this 10-core ceiling comes from macOS scheduler policy (keeping one 5-core P-group uncongested for OS, GUI, and driver threads on a chip that has no E-cores), CoreAudio's `os_workgroup` sizing, or Live's worker-thread pool is an open question—but in practice, the 15-core M5 Pro behaves as a **10-core audio processor** in Ableton Live 12.

---

## Tracks 12–16: How 10 Active Cores Handle 15 and 16 Tracks

Following the CPU History graphs from 11 to 16 tracks reveals how the 10 active cores (5 Super + 5 P) distribute 2:1 track pairs:

![15 tracks Echo at 32 samples](images/15ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20Echo%2042.png)\
<sub>*15 tracks, 32 samples, Ableton Echo (captured at 42/track; final CSV limit 43/track = 645 total): Super Cores 11–15 run near full height (carrying 2 tracks each), while P-cores 6–10 sit flat around ~60% height (carrying 1 track each).*</sub>

![15 tracks Waves L4 at 32 samples](images/15ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2017.png)\
<sub>*15 tracks, 32 samples, Waves L4 (17/track = 255 total): audio runs on P-cores 1–5 (~75–85% height) and Super Cores 11–15 (near 100%), with P-cores 6–10 idle/background.*</sub>

At **15 tracks** on 10 active cores, five cores have to run **2 tracks each** ($5 \times 2 = 10$ tracks) while the other five cores run **1 track each** ($5 \times 1 = 5$ tracks):
- In the 15-track Echo capture (`15ch Echo 42.png`), **Super Cores 11–15 are pinned near the top** ($2 \times 43 = 86$ Echo instances per Super Core, right at the 5-track Super Core limit of 84), while **P-cores 6–10 sit flat at ~60% bar height** because each P-core is only running a single track of 43 instances (well below a P-core's ~75-instance capacity). As a result, 15 tracks of Echo (**645 total**) still falls **$-14.0\%$ short of the 10-track peak (750)**.
- On **Waves L4**, 15 tracks at 17/track yields **255 total instances**, where $2 \times 17 = 34$ per Super Core and $1 \times 17 = 17$ per P-core ($\sim 75\text{–}85\%$ bar height on Cores 1–5). Since $\pm 1$ instance per track across 15 tracks equals $\pm 15$ total instances, 255 vs. 260 ($-1.9\%$) is effectively a **tie with the 10-track peak** within measurement resolution.

Then comes **Track 16**:

![16 tracks Waves L4 at 32 samples](images/16ch%20macOS%20MacMini%20M5Pro%20Babyface@32%20WavesL4%2014.png)\
<sub>*16 tracks, 32 samples, Waves L4 (14/track = 224 total): with 16 tracks on 10 cores, a 6th core—at least one mid-tier P-core—must now carry 2 tracks. P-cores 1–5 jump to ~90% height while Super Cores 11–15 drop lower, triggering a second L4 drop.*</sub>

With **16 tracks** sharing 10 cores, **six cores must carry 2 tracks each** ($6 \times 2 + 4 \times 1 = 16$). Because there are only five Super Cores, **at least one mid-tier P-core is now forced to run 2 tracks in series**:
- On **Waves L4**, where the P-cores are noticeably slower than the Super Cores, forcing a P-core to carry 2 tracks triggers a **second cliff**: per-track headroom drops **$-17.6\%$** (from 17 down to **14/track**), and total instances fall **$-12.2\%$** (from 255 down to **224 total**). You can see this in `16ch WavesL4 14.png`: P-cores 1–5 climb to ~90% height while Super Cores 11–15 drop back to ~70–75%.
- On **Ableton Echo**, where P-core throughput is much closer to Super Core throughput, stepping from 15 to 16 tracks reduces per-track count by only **$-4.7\%$** (43 down to **41/track**), and total instances tick up slightly from **645 to 656 ($+1.7\%$)**. From 17 to 20 tracks (as the remaining four active P-cores take on a second track), per-track limits should hold near ~41 Echo and ~14 L4.

### Summary: The 4 Scaling Regimes of the 15-Core M5 Pro

| Regime | Channels | Active Audio Cores (Observed in Capture vs. Inferred) | Echo / Trk (Total) @ 32 | Echo / Trk (Total) @ 128 | Waves L4 / Trk (Total) @ 32 | What Happens |
|:---|:---:|:---|:---:|:---:|:---:|:---|
| **1. Super Cores Only (1:1)** | **1 ch** | **Observed:** Echo on Core 11 (~80%) + Core 12; L4 rotates 11–15 | 90 (90) | 97 (97) | 44 (44) | Single-track Super Core load. |
| | **2 ch** | **Observed (L4):** Super Cores 11–15 | 87 (174) | 91 (182) | 37 (74) | Both tracks stay on Super Cores (no heavy P-core load). |
| | **3 ch** | *Inferred from per-track numbers:* 3 S-Cores | 86 (258) | — | 35 (105) | Interpolated between 2ch and 5ch captures. |
| | **4 ch** | *Inferred from per-track numbers:* 4 S-Cores | 86 (344) | — | 32 (128) | Interpolated between 2ch and 5ch captures. |
| | **5 ch** | **Observed:** All 5 Super Cores (11–15) | **84 (420)** | 90 (450) | **31 (155)** | All 5 Super Cores loaded; no heavy load on P-cores 1–10. |
| **2. P-Core Spill (1:1)** | **6 ch** | *Inferred from per-track numbers:* 5 S + 1 P | **81** (486) | — | **29** (174) | First spill to P-cores; slowest core sets session limit. |
| | **7 ch** | *Inferred from per-track numbers:* 5 S + 2 P | — | — | **29** (203) | Flat P-Core Plateau (29 L4/trk). |
| | **8 ch** | *Inferred from per-track numbers:* 5 S + 3 P | **81** (648) | — | **29** (232) | Flat P-Core Plateau (81 Echo / 29 L4). |
| | **9 ch** | **Observed:** 9 Cores (P-Cores 6–9 + S-Cores 11–15) | 77 (693) | — | 27 (243) | 9 cores active; Core 10 idle; Cores 1–5 background. |
| | **10 ch** | **Observed:** 10 Cores (P-Cores 6–10 + S-Cores 11–15) | **75 (750 PEAK)** | **74 (740)** | **26 (260 PEAK)** | **Max 1:1 Saturation (10 active cores; Cores 1–5 background).** |
| **3. Cliff #1 (11–15 ch)** | **11 ch** | **Observed:** 10 Cores (Cores 6–15 active; 1–5 background) | **45 (495)** | **47 (517)** | **19 (209)** | **10-Core Cliff:** 11 tracks share 10 cores ($-40\%$ Echo, $-27\%$ L4). |
| | **12 ch** | **Observed:** 10 Cores (Echo: 1–5 + 11–15; L4: 6–15) | 45 (540) | — | 18 (216) | 12 tracks share 10 cores; P-groups 1–5 & 6–10 interchangeable. |
| | **15 ch** | **Observed:** 10 Cores (Echo: 6–15, P ~60%; L4: 1–5 + 11–15, P ~75–85%) | 43 (645) | — | 17 (255) | 15 tracks on 10 cores: L4 ties 10ch peak; Echo $-14\%$ below peak. |
| **4. 16 Channels** | **16 ch** | **Observed (L4):** 10 Cores (P 1–5 ~90% + 11–15); *Echo inferred* | **41 (656)** | — | **14 (224)** | **L4 Cliff #2** ($-17.6\%\text{/trk}$, $-12.2\%\text{ total}$); **Echo** $+1.7\%\text{ total}$. |

---

## Buffer Inversion on M5 Pro: Single-Track and 5-Track Sessions

In Part 1 on the M6, raising the buffer from 32 to $\ge 64$ samples on a single Waves L4 track demoted the audio thread from the Super Cores onto a single mid-tier P-core, cutting headroom by 27%.

The M5 Pro reproduces that exact single-track buffer inversion—and exposes a **multi-track version** at 5 channels:

![1 track Waves L4 at 64 samples](images/01ch%20macOS%20MacMini%20M5Pro%20Babyface@64%20WavesL4%2034.png)\
<sub>*1 track, 64 samples, Waves L4 (34 instances): switching from 32 to 64 samples pulls the thread off Super Cores 11–15 (now idle) and pins it to Core 1 — Performance, dropping capacity by 22.7% (44 → 34).*</sub>

- **Single-Track Buffer Scaling (32 → 64 → 128 → 256 samples):**
  - **Ableton Echo (1 track):** Scales upward normally with no buffer inversion: **90 → 93 → 97 → 99 instances** ($+10.0\%$ from 32 to 256 samples).
  - **Waves L4 (1 track):** Drops **$-22.7\%$ at 64 samples** (**44 → 34 instances**) as the thread leaves Super Cores 11–15 and pins `Core 1 — Performance` at 100%. Raising the buffer further to 128 and 256 samples only partially recovers to **36 and 37 instances**—still $15.9\%$ below the 32-sample result.

What about a **5-track session**? At 32 samples (`05ch WavesL4 31.png` shown earlier), all 5 tracks of Waves L4 stayed on the 5 Super Cores (Cores 11–15) and achieved 31 instances/track. When the buffer is raised to 64 and 128 samples, per-track counts rise slightly to **33 and 36 instances/track**—which at first glance looks like normal buffer scaling.

Then you open Activity Monitor at 128 samples:

![5 tracks Waves L4 at 128 samples](images/05ch%20macOS%20MacMini%20M5Pro%20Babyface@128%20WavesL4%2036.png)\
<sub>*5 tracks, 128 samples, Waves L4 (36/track = 180 total): instead of running all 5 tracks on Super Cores 11–15 as it did at 32 samples, one track is pinned near 100% on Core 1 — Performance while Core 11 — Super sits only ~50% loaded.*</sub>

At 128 samples, **one of the five tracks gets pulled onto `Core 1 — Performance` (pinned near 100%)**, while `Core 11 — Super` sits roughly half-loaded (~50%) and Super Cores 12–15 carry the other four tracks. Notice the instance count: **36 instances per track**—the exact single-track ceiling of `Core 1 — Performance` at 128 samples (36 instances). Because that one P-core hits its limit at 36 instances, the entire 5-track session is capped at 36/track, even though the Super Cores could handle well over 40 instances per track at 128 samples.

---

## Cross-System Comparison: M6 (12c) vs. M5 Pro (15c) vs. Intel 270K Plus (8P)

Putting all three measured systems side by side shows where each topology wins—and where the M5 Pro's 10-core audio ceiling changes the picture.

### Table 1: Waves L4 Ultramaximizer (32 Samples Unless Noted)

| Tracks | Mac mini M6 (12c: 2S + 4P + 6E) | Mac mini M5 Pro (15c: 5S + 10P) | Intel Core Ultra 7 270K Plus (8P) | Key Takeaway |
|:---:|:---:|:---:|:---:|:---|
| **1** | **52** | **44** | 36 (Babyface @ 48); PCIe n/t | M6 Super Core leads M5 Pro by $+18.2\%$. |
| **2** | **37/trk (74)** | **37/trk (74)** | n/t | Tied at 74 total (M6 via P-spill; M5 Pro on S-cores). |
| **5** | **35/trk (175)** | **31/trk (155)** | n/t | M6 leads by $+12.9\%$ as M5 Pro L4 drops across 5 S-cores. |
| **6** | **35/trk (210 PEAK)** | **29/trk (174)** | n/t | M6's 6 fast cores beat M5 Pro at 6 tracks by **$+20.7\%$**. |
| **8** | n/t | **29/trk (232)** | **36/trk (288, PCIe @ 32)**<br>30/trk (240, Babyface @ 48) | Intel's 8 uniform P-cores beat M5 Pro at 8 tracks ($+24.1\%$ PCIe, $+3.4\%$ USB). |
| **10** | n/t | **26/trk (260 PEAK)** | n/t | M5 Pro 1:1 peak (+23.8% over M6 6ch uniform; $-9.7\%$ behind Intel 8ch PCIe). |
| **11** | n/t | **19/trk (209)** | n/t | **M5 Pro Cliff #1:** 11th track forces 2:1 sharing on 10 cores. |
| **12** | 17/trk (204) | **18/trk (216)** | n/t | Because of the 10-core cap, 15c M5 Pro leads 12c M6 by only $+5.9\%$. |
| **15** | n/t | **17/trk (255)** | n/t | Ties the 10-track peak (260) within $\pm 1\text{/trk}$ ($\pm 15$) test resolution. |
| **16** | n/t | **14/trk (224)** | n/t | **M5 Pro Cliff #2:** 6th core (a P-core) takes a 2nd track ($-12.2\%$ total). |

### Table 2: Ableton Echo (32 Samples Unless Noted)

| Tracks | Mac mini M6 (12c: 2S + 4P + 6E) | Mac mini M5 Pro (15c: 5S + 10P) | Intel Core Ultra 7 270K Plus (8P) | Key Takeaway |
|:---:|:---:|:---:|:---:|:---|
| **1** | **102** | **90** | **89** (PCIe @ 32)<br>82 (Babyface @ 48) | M6 leads single-track (+13.3%); M5 Pro and Intel PCIe tie (90 vs. 89). |
| **3 – 5** | 3ch: 85/trk (255)<br>4ch: 84/trk (336)<br>5ch: 83/trk (415) | **3ch: 86/trk (258)**<br>**4ch: 86/trk (344)**<br>**5ch: 84/trk (420)** | n/t | M5 Pro's **5 Super Cores** hold 1:1 load and edge out the M6 across 3–5 tracks. |
| **6** | **85/trk (510 PEAK)** *(83/trk = 498 earlier)* | **81/trk (486)** | n/t | M5 Pro spills to P-core on Track 6, falling slightly behind M6. |
| **8** | n/t | **81/trk (648)** | **87/trk (696, PCIe @ 32)**<br>80/trk (640, Babyface @ 48) | At 8 tracks, M5 Pro edges out Intel on Babyface (648 vs. 640), trails PCIe (696). |
| **10** | n/t | **75/trk (750 PEAK)** | n/t | **M5 Pro 1:1 Peak:** 10 active cores beat Intel's 8-track peak (+7.8% vs. PCIe, +17.2% vs. USB). |
| **11** | n/t | **45/trk (495)** | n/t | **M5 Pro Cliff #1:** $-40\%$ per-track drop as 11 tracks share 10 cores. |
| **12** | 42/trk (504) | **45/trk (540)** | n/t | M5 Pro leads M6 by $+7.1\%$ at 12 tracks. |
| **15** | n/t | **43/trk (645)** | n/t | 5 Super Cores run 2 tracks, 5 P-cores run 1 track ($-14.0\%$ below 10ch peak). |
| **16** | n/t | **41/trk (656)** | **43/trk (688, PCIe @ 32)**<br>**41/trk (656, Babyface @ 48)** | At 16 tracks, the 15-core M5 Pro **ties** the 8-core Intel 270K Plus on Babyface (656)! |

---

## Key Takeaways: What the M5 Pro Data Shows

1. **Five Super Cores fix the M6's Track 2 P-core spill at 32 samples—especially for native devices.**
   On the M6, having only two Super Cores meant Track 2 immediately fell onto a slower P-core. On the M5 Pro, tracks 1 through 5 stay on Super Cores 11–15 at 32 samples. With native Ableton Echo, that gives you nearly linear Super Core scaling up to 5 tracks (90 down to 84 per track). With third-party Waves L4, however, intra-group contention or plugin overhead across the five Super Cores still costs $29.5\%$ per track by Track 5 (44 down to 31) before a single P-core is even touched.
2. **In Ableton Live 12, the 15-core M5 Pro operates as a 10-core audio chip.**
   Because the M5 Pro has no dedicated Efficiency cores, its 15 cores behave in three 5-core groups (P-cores 1–5, P-cores 6–10, and Super Cores 11–15). In every multi-track test up to 16 channels, one of the two P-core groups is always left to handle low-level OS/background tasks while real-time audio is restricted to the remaining **10 cores (5 Super + 5 P)**.
3. **10 tracks is the true sweet spot; Track 11 is a steep cliff.**
   At 10 tracks (one track per active audio core), the M5 Pro hits its highest aggregate capacity (**750 Echo / 260 Waves L4**), beating the M6's 6-track peak by $+47\%$ on Echo and $+24\%$ on L4. But adding an 11th track forces 2:1 track sharing on the 10 active cores rather than waking up the 5 idle P-cores, causing an immediate $-40\%$ per-track drop on Echo and $-27\%$ on L4. By 16 tracks on Echo, the 15-core M5 Pro (41/track = 656) ends up tied with the 8-P-core Intel 270K Plus on the same Babyface interface (41/track = 656).
4. **Buffer inversion can bottleneck multi-track sessions too.**
   Raising the buffer from 32 to $\ge 64$ samples still demotes a single third-party plugin track from the Super Cores to a P-core ($-22.7\%$ at 64 samples). Even more surprisingly, in a 5-track Waves L4 session at 128 samples, one of the five tracks gets pinned to `Core 1 — Performance` while `Core 11 — Super` sits half-empty—capping all five tracks at the P-core's 36-instance limit.

Just as in Part 1, none of this means a 15-core M5 Pro will struggle in normal day-to-day music production—750 Echo instances or 260 mastering limiters at a 32-sample buffer is an enormous amount of real-time DSP. What these benchmarks and Activity Monitor captures do show is how Apple's multi-tier core groupings and macOS thread scheduling shape the real-time ceiling under the hood: you get 5 Super Cores for up to 5 heavy tracks, 5 additional P-cores that carry you to a 10-track peak, and a third 5-core P-group that stays in reserve for the OS rather than scaling your mix from 11 to 15 tracks.

[← Back to Part 1 (Mac mini M6)](https://camplaix.github.io/beyond-p-cores-apple-m6-ableton-live-12/) | [← Back to camplaix.github.io](https://camplaix.github.io/)