# **Ableton Live 12 Multi-Core Scaling Benchmark Archive**

## **Overview & Methodology**

* **Scope:** Benchmark measurements and macOS Activity Monitor captures investigating real-time audio multi-core scaling in Ableton Live 12.

* **Methodology:** Every track in a project carries the same plugin count. Instances are raised (or removed) on all tracks equally until audible dropouts occur. The recorded value is the highest per-track count without dropouts.

* **Filename syntax:** `[Channels] + [OS] + [CPU] + [Interface]@[Buffer] + [Effect] + [Max Instances Per Track].als`

* **Sample rate:** $44.1\text{ kHz}$ on all systems.

## **Test Environment**

1. **Systems tested**

   * **Mac mini M6 (macOS Golden Gate, 24 GB):** 12 cores. In Activity Monitor's numbering: Cores 1–6 = Efficiency, Cores 7–8 = Super Cores, Cores 9–12 = mid-tier Performance cores.

   * **Mac mini M5 Pro (macOS Golden Gate, 24 GB):** 15 cores (no Efficiency cores). In Activity Monitor's numbering: Cores 1–10 = mid-tier Performance cores, Cores 11–15 = Super Cores. Across the captures, Activity Monitor's load patterns consistently divide into three 5-core groups: Cores 1–5 (P-cores), Cores 6–10 (P-cores), and Cores 11–15 (Super Cores).

   * **Intel Core Ultra 7 270K Plus (Windows 11 Pro 25H2):** 8 Lion Cove P-cores used for audio; Live restricts its real-time audio (MMCSS) threads to the P-cores. DDR5 CUDIMM $8800\text{ MT/s}$. C-States off, turbo ratios locked at $5.4\text{ GHz}$ (P-cores) and $4.7\text{ GHz}$ (E-cores), Ultimate Performance power plan, Virtualization-based Security / Core Isolation disabled.

2. **Audio interfaces**

   * **RME Babyface Pro FS (USB 2.0):** macOS CoreAudio at 32 samples; Windows ASIO at 48–64 samples (48 is the driver's minimum on Windows).

   * **RME HDSPe AIO Pro (PCIe):** Windows ASIO at 32–64 samples, as a reference without USB driver overhead. Also tried on the Mac in a Thunderbolt PCIe enclosure; no better than the Babyface, so all Mac tests use the Babyface.

3. **Drivers:** Latest RME DriverKit driver on macOS (the older kext driver gave the same results); latest RME ASIO drivers on Windows.

4. **Plugins:** Ableton Echo (native) and Waves L4 Ultramaximizer (third-party). FabFilter Saturn was used to confirm the buffer inversion effect (AUv2 and VST3).

## **Dataset: Transcribed Project Catalog (CSV)**

All values are actual per-track counts, with every track loaded at the same time. **Note: The CSV table below is the authoritative record of final dropout-free maximums.** Where an Activity Monitor screenshot filename shows a slightly lower instance count (specifically `09ch Echo 73` vs. `77` in the CSV, and `15ch Echo 42` vs. `43` in the CSV), the capture was saved before the count was raised to its final dropout limit, and the CSV value prevails.

```
Channels,OS,CPU,Audio_Interface,Buffer_Size,Effect,Max_Instances_Per_Track,Total_Instances
1,macOS,MacMini M5Pro,Babyface,32,Echo,90,90
1,macOS,MacMini M5Pro,Babyface,32,WavesL4,44,44
1,macOS,MacMini M5Pro,Babyface,64,Echo,93,93
1,macOS,MacMini M5Pro,Babyface,64,WavesL4,34,34
1,macOS,MacMini M5Pro,Babyface,128,Echo,97,97
1,macOS,MacMini M5Pro,Babyface,128,WavesL4,36,36
1,macOS,MacMini M5Pro,Babyface,256,Echo,99,99
1,macOS,MacMini M5Pro,Babyface,256,WavesL4,37,37
1,macOS,MacMini M6,Babyface,32,Echo,102,102
1,macOS,MacMini M6,Babyface,32,WavesL4,52,52
1,macOS,MacMini M6,Babyface,64,Echo,107,107
1,macOS,MacMini M6,Babyface,64,WavesL4,38,38
1,macOS,MacMini M6,Babyface,128,Echo,109,109
1,macOS,MacMini M6,Babyface,128,WavesL4,40,40
1,macOS,MacMini M6,Babyface,256,Echo,110,110
1,macOS,MacMini M6,Babyface,256,WavesL4,42,42
1,win11,Core Ultra 7 270K Plus,AIO,32,Echo,89,89
1,win11,Core Ultra 7 270K Plus,AIO,64,Echo,90,90
1,win11,Core Ultra 7 270K Plus,Babyface,48,Echo,82,82
1,win11,Core Ultra 7 270K Plus,Babyface,48,WavesL4,36,36
1,win11,Core Ultra 7 270K Plus,Babyface,64,Echo,88,88
2,macOS,MacMini M5Pro,Babyface,32,Echo,87,174
2,macOS,MacMini M5Pro,Babyface,32,WavesL4,37,74
2,macOS,MacMini M5Pro,Babyface,64,Echo,88,176
2,macOS,MacMini M5Pro,Babyface,128,Echo,91,182
2,macOS,MacMini M5Pro,Babyface,256,Echo,93,186
2,macOS,MacMini M6,Babyface,32,Echo,88,176
2,macOS,MacMini M6,Babyface,32,WavesL4,37,74
2,macOS,MacMini M6,Babyface,256,WavesL4,42,84
3,macOS,MacMini M5Pro,Babyface,32,Echo,86,258
3,macOS,MacMini M5Pro,Babyface,32,WavesL4,35,105
3,macOS,MacMini M6,Babyface,32,Echo,85,255
4,macOS,MacMini M5Pro,Babyface,32,Echo,86,344
4,macOS,MacMini M5Pro,Babyface,32,WavesL4,32,128
4,macOS,MacMini M6,Babyface,32,Echo,84,336
5,macOS,MacMini M5Pro,Babyface,32,Echo,84,420
5,macOS,MacMini M5Pro,Babyface,32,WavesL4,31,155
5,macOS,MacMini M5Pro,Babyface,64,Echo,88,440
5,macOS,MacMini M5Pro,Babyface,64,WavesL4,33,165
5,macOS,MacMini M5Pro,Babyface,128,Echo,90,450
5,macOS,MacMini M5Pro,Babyface,128,WavesL4,36,180
5,macOS,MacMini M6,Babyface,32,Echo,83,415
5,macOS,MacMini M6,Babyface,32,WavesL4,35,175
5,macOS,MacMini M6,Babyface,256,WavesL4,41,205
6,macOS,MacMini M5Pro,Babyface,32,Echo,81,486
6,macOS,MacMini M5Pro,Babyface,32,WavesL4,29,174
6,macOS,MacMini M6,Babyface,32,Echo,85,510
6,macOS,MacMini M6,Babyface,32,WavesL4,35,210
6,macOS,MacMini M6,Babyface,64,Echo,86,516
6,macOS,MacMini M6,Babyface,64,WavesL4,37,222
6,macOS,MacMini M6,Babyface,128,Echo,89,534
6,macOS,MacMini M6,Babyface,128,WavesL4,39,234
6,macOS,MacMini M6,Babyface,256,Echo,90,540
6,macOS,MacMini M6,Babyface,256,WavesL4,40,240
7,macOS,MacMini M5Pro,Babyface,32,WavesL4,29,203
8,macOS,MacMini M5Pro,Babyface,32,Echo,81,648
8,macOS,MacMini M5Pro,Babyface,32,WavesL4,29,232
8,win11,Core Ultra 7 270K Plus,AIO,32,Echo,87,696
8,win11,Core Ultra 7 270K Plus,AIO,32,WavesL4,36,288
8,win11,Core Ultra 7 270K Plus,AIO,64,Echo,88,704
8,win11,Core Ultra 7 270K Plus,Babyface,48,Echo,80,640
8,win11,Core Ultra 7 270K Plus,Babyface,48,WavesL4,30,240
8,win11,Core Ultra 7 270K Plus,Babyface,64,Echo,87,696
9,macOS,MacMini M5Pro,Babyface,32,Echo,77,693
9,macOS,MacMini M5Pro,Babyface,32,WavesL4,27,243
10,macOS,MacMini M5Pro,Babyface,32,Echo,75,750
10,macOS,MacMini M5Pro,Babyface,32,WavesL4,26,260
10,macOS,MacMini M5Pro,Babyface,128,Echo,74,740
11,macOS,MacMini M5Pro,Babyface,32,Echo,45,495
11,macOS,MacMini M5Pro,Babyface,32,WavesL4,19,209
11,macOS,MacMini M5Pro,Babyface,128,Echo,47,517
12,macOS,MacMini M5Pro,Babyface,32,Echo,45,540
12,macOS,MacMini M5Pro,Babyface,32,WavesL4,18,216
12,macOS,MacMini M6,Babyface,32,Echo,42,504
12,macOS,MacMini M6,Babyface,32,WavesL4,17,204
12,macOS,MacMini M6,Babyface,256,Echo,46,552
12,macOS,MacMini M6,Babyface,256,WavesL4,20,240
15,macOS,MacMini M5Pro,Babyface,32,Echo,43,645
15,macOS,MacMini M5Pro,Babyface,32,WavesL4,17,255
16,macOS,MacMini M5Pro,Babyface,32,Echo,41,656
16,macOS,MacMini M5Pro,Babyface,32,WavesL4,14,224
16,win11,Core Ultra 7 270K Plus,AIO,32,Echo,43,688
16,win11,Core Ultra 7 270K Plus,AIO,64,Echo,45,720
16,win11,Core Ultra 7 270K Plus,Babyface,48,Echo,41,656
16,win11,Core Ultra 7 270K Plus,Babyface,64,Echo,44,704
```

**Additional result not saved as a project file:** M6, 6 tracks, Waves L4, 32 samples, asymmetric load: 50 instances on each of the two Super Core tracks and 37 on each of the four P-core tracks = 248 total. The Super Core tracks were found by trial and error (raising one track at a time while listening for dropouts, with Activity Monitor open).

## **macOS Activity Monitor Captures (Mac mini M6)**

All CPU History captures of the 1-, 2- and 6-track sessions show **Ableton Echo**. The buffer inversion capture shows **Waves L4**.

Activity Monitor averages each bar over about a second, threads move between cores, and dropouts are caused by the slowest single callback. A core at its dropout limit therefore does not show $100\%$.

* **1 track (32 samples):** The load alternates between Super Cores 7 and 8 (only one runs the thread at any moment). P-cores 9–12 stay idle.

* **2 tracks (32 samples):** The second track's load goes to the P-cores, mainly Core 9, not to the spare Super Core time. No core is pinned at $100\%$; the Super Cores together carry about one track's worth of work and the rest runs on P-cores.

* **6 tracks (32 samples):** Cores 7–12 (both Super Cores and all four P-cores) run near their limit. E-cores 1–6 stay near idle, as expected at 32 samples.

* **Buffer inversion (Waves L4, 1 track, $32 \rightarrow 512$ samples):** At 32 samples the thread alternates between Super Cores 7 and 8. From 64 samples it moves to a single P-core (Core 9). E-cores stay idle at every buffer size.

### 1-Channel Load (32 samples, Echo)

*\[Refer to attached image: `1-Channel Load (32 samples, Echo).gif`\]*

### 2-Channel Load (32 samples, Echo)

*\[Refer to attached image: `2-Channel Load (32 samples, Echo).gif`\]*

### 6-Channel Load (32 samples, Echo — 1:1 Fast-Core Saturation)

*\[Refer to attached image: `6-Channel Load (32 samples, Echo).gif`\]*

## **Historical Reference Benchmarks**

* **Effect:** Ableton Live 12 Echo

* **Audio interface:** RME Babyface Pro FS

* **Systems:**

  * Intel Core Ultra 5 245K (6 P + 8 E): **limited to 4 P-cores for all channel counts**, ratios locked at $5.0\text{ GHz}$, Windows, **48 samples** (the Babyface's Windows minimum)

  * AMD Ryzen 7 9800X3D (8 cores, SMT off): ratios locked at $5.0\text{ GHz}$, C-states off, Windows, **48 samples**

  * Apple MacBook Pro base M5 (4 identical fast P-cores + 6 E-cores, up to $4.6\text{ GHz}$): default settings, macOS, **32 samples**

* **Method:** Same as above: every track carries the same instance count, raised on all tracks equally until dropouts.

### **Important: the 8- and 16-channel values are normalized**

Where a system had more tracks than fast cores, the table shows the **actual per-track result multiplied by the number of tracks sharing each core**, so the values read as "per fast core". The 1- and 4-channel values are actual per-track counts (every system had at least one fast core per track).

| System | Fast cores used | 8 channels | 16 channels | 
| ----- | ----- | ----- | ----- | 
| 245K | 4 | $\text{actual} \times 2$ | $\text{actual} \times 4$ | 
| 9800X3D | 8 | actual (1:1) | $\text{actual} \times 2$ | 
| Base M5 | 4 | $\text{actual} \times 2$ | $\text{actual} \times 4$ | 

### **Echo instances (1- and 4-channel values are actual; 8- and 16-channel values are normalized per fast core)**

**Lowest buffer (245K and 9800X3D at 48 samples; M5 at 32 samples)**

| Channels | 245K (4 P-cores, $5.0\text{ GHz}$) | 9800X3D ($5.0\text{ GHz}$) | Base M5 ($4.6\text{ GHz}$) | 
| ----- | ----- | ----- | ----- | 
| 1 | 75 | 73 | 96 | 
| 4 | 75 | 73 | 90 | 
| 8 (normalized) | 76 | 74 | 92 | 
| 16 (normalized) | 76 | 72 | 96 | 

**64 samples**

| Channels | 245K (4 P-cores, $5.0\text{ GHz}$) | 9800X3D ($5.0\text{ GHz}$) | Base M5 ($4.6\text{ GHz}$) | 
| ----- | ----- | ----- | ----- | 
| 1 | 84 | 85 | 99 | 
| 4 | 82 | 85 | 91 | 
| 8 (normalized) | 84 | 84 | 97 | 
| 16 (normalized) | 84 | 84 | 96 | 

### **Values to double-check against the original Live project**

* **245K, 16 channels, lowest buffer:** this archive says **76**; the project screenshot appears to show **75** in that row.

* **245K and base M5 at 64 samples:** not visible in the project screenshot; values are from the earlier archive.

## **Mac mini M6 vs. M5 Pro vs. Intel 270K Plus**

### **Description**

Comparison tables and analysis of Waves L4 and Ableton Echo instances before dropouts at 32 samples in Ableton Live 12 across the measured systems: Mac mini M6 (12-core), Mac mini M5 Pro (15-core), and Intel Core Ultra 7 270K Plus (8 Lion Cove P-cores). All tests use a symmetric plugin load across tracks.

### **Summary of Key Observations**

* **Single track (32 samples):** The M6 Super Core leads (52 L4, 102 Echo). The 15-core M5 Pro Super Core trails the M6 by $-15.4\%$ on L4 (44) and $-11.8\%$ on Echo (90). In the 1-track captures, Waves L4 (`01ch WavesL4 44.png`) rotates its load across all five Super Cores (Cores 11–15), whereas Ableton Echo (`01ch Echo 90.png`) sits mostly on Core 11 ($\sim 80\%$) with secondary load on Core 12 ($\sim 25\%$). Intel's single-track results are 36 L4 (Babyface @ 48) and 89 Echo (PCIe @ 32) — virtually tied with the M5 Pro on single-track Echo.

* **Super Core retention vs. Multi-Track Scaling (Tracks 1–5 at 32 samples):** Captures taken at 1, 2, and 5 tracks (`01ch`, `02ch WavesL4 37.png`, `05ch Echo 84.png`, and `05ch WavesL4 31.png`, with Tracks 3 and 4 interpolated from their per-track counts) show that at 32 samples, **heavy audio load stays on the five Super Cores (Cores 11–15) through 5 tracks**, with no heavy load on P-cores 1–10 (only light background/kernel activity on one 5-core P-group while the other 5-core P-group sits near zero). Even while staying on the Super Cores, however, the two plugins scale differently as more tracks are added:

  * **Ableton Echo** retains $93.3\%$ of its single-track performance across 5 tracks ($90 \rightarrow 87 \rightarrow 86 \rightarrow 86 \rightarrow 84\text{/track}$, $420\text{ total}$).

  * **Waves L4** loses $-15.9\%$ at 2 tracks ($44 \rightarrow 37\text{/track}$, while three Super Cores sit idle) and $-29.5\%$ across 5 tracks ($44 \rightarrow 37 \rightarrow 35 \rightarrow 32 \rightarrow 31\text{/track}$, $155\text{ total}$). Because Echo loses only $-3.3\%$ at 2 tracks and $-6.7\%$ at 5 tracks on the same Super Cores, this penalty is **Waves L4-specific, cause unknown** (possibilities include shared cache or memory bandwidth contention, plugin-internal shared state, or thread migration across all five Super Cores).

* **P-Core Spill & Plateau (Tracks 6–10):** Per-track numbers indicate that audio load spills onto the mid-tier P-cores starting at **Track 6**, establishing a flat per-track plateau across Tracks 6–8 at **81/track on Echo** (486 at 6ch, 648 at 8ch) and **29/track on Waves L4** (174 at 6ch, 203 at 7ch, 232 at 8ch) before stepping down slightly at 9–10 tracks ($77 \rightarrow 75$ Echo; $27 \rightarrow 26$ L4), where the 9- and 10-track captures confirm 4 and 5 active P-cores alongside the 5 Super Cores. At 6 tracks of Waves L4, the 12-core M6 (210 total @ 35/track) beats the 15-core M5 Pro (174 total @ 29/track) by $+20.7\%$.

* **Observed 10-Core Audio Ceiling & Interchangeable P-Groups on M5 Pro:** Across every multi-track capture from 9 to 16 channels, **no more than 10 of the M5 Pro's 15 cores ever carry heavy audio load simultaneously** — always the 5 Super Cores (11–15) plus one 5-core P-group (either Cores 6–10 or Cores 1–5).

  * **Swap between P-Groups:** Most multi-track captures place heavy audio load on **Cores 6–10** while **Cores 1–5** carry low-level background/system (red) activity. However, three multi-track captures — `12ch Echo 45.png`, `15ch WavesL4 17.png`, and `16ch WavesL4 14.png` — reverse this, placing heavy audio load on **Cores 1–5** while **Cores 6–10** carry the low-level background/red activity. In every capture, background (red) kernel load sits in whichever P-group is not carrying audio.

  * **Interchangeable Performance:** Both P-groups deliver identical audio headroom: `11ch Echo 45` (audio on Cores 6–10) and `12ch Echo 45` (audio on Cores 1–5) both max out at exactly 45 instances/track.

  * **Open Question on Cause:** Whether this 10-core ceiling is driven by macOS scheduler policy (e.g., keeping one 5-core P-group free for OS/background work in the absence of E-cores), `os_workgroup` behavior, or Live's internal worker-thread pool is unknown, but the observed limit is strictly 10 cores across all tests.

* **Peak Capacity (10 tracks) & Behaviour at 11–16 Tracks:**

  * **10-Track Peak (1:1 Saturation):** With 10 cores taking audio load, the M5 Pro reaches **260 L4** (26/track) and **750 Echo** (75/track) at 10 tracks. Expanding the buffer from 32 to 128 samples at 10 tracks gives Echo no gain ($75\text{/track}$ at 32 samples vs. $74\text{/track}$ at 128 samples).

  * **Track 11 Cliff (11–15 Tracks):** Adding an 11th track does not recruit the 5 remaining P-cores; all 11 tracks share the 10 active cores (forcing 2:1 track sharing on one core). Per-track limits drop by $-26.9\%$ on L4 ($26 \rightarrow 19\text{/track}$, $209\text{ total}$) and $-40.0\%$ on Echo ($75 \rightarrow 45\text{/track}$, $495\text{ total}$; and $74 \rightarrow 47\text{/track}$ at 128 samples). Across the 11- to 15-track captures, the 5 Super Cores run near full height while the 5 active P-cores run lower ($\sim 70\%$ bar height in `11ch Echo 45`, $\sim 60\%$ in `15ch Echo 42`, and $\sim 75\text{–}85\%$ in `15ch WavesL4 17`), consistent with the 5 Super Cores taking the 2:1 track pairs while the 5 P-cores carry 1 track each. At 15 tracks, **Echo reaches 645 total (43/track)** — still $-14.0\%$ below its 10-track peak of 750 — whereas **Waves L4 reaches 255 total (17/track)**, which is within $-1.9\%$ of its 260 peak (effectively a tie given the $\pm 1\text{ instance/track} = \pm 15\text{ total}$ resolution at 15 tracks).

  * **Track 16 Divergence (L4 Cliff #2 vs. Echo Plateau):** At 16 tracks, 6 of the 10 active cores must carry 2 tracks each, meaning at least one mid-tier P-core must now process 2 tracks. On **Waves L4** (`16ch WavesL4 14.png`, where active P-cores 1–5 rise to $\sim 90\%$ bar height), this triggers a second sharp drop of $-17.6\%$ per track ($17 \rightarrow 14\text{/track}$) and $-12.2\%$ in total instances ($255 \rightarrow 224$). On **Ableton Echo**, however, per-track count steps down by only $-4.7\%$ ($43 \rightarrow 41\text{/track}$) while total instances rise slightly by $+1.7\%$ ($645 \rightarrow 656$). Assuming this pattern continues until all 5 active P-cores carry 2 tracks each, 17–20 tracks should hold near $\sim 41\text{/track}$ on Echo and $\sim 14\text{/track}$ on L4.

* **Single-Track & Multi-Track Buffer Inversion on M5 Pro:**

  * **Native Echo (No Buffer Inversion):** Echo scales upward normally on a single track as the buffer increases ($90 \rightarrow 93 \rightarrow 97 \rightarrow 99$ instances across $32 \rightarrow 64 \rightarrow 128 \rightarrow 256$ samples).

  * **1-Track Waves L4 Buffer Inversion:** Raising the buffer from 32 to 64 samples on a single Waves L4 track (`01ch @ 64 WavesL4 34.png`) moves the audio thread off the Super Cores (Cores 11–15 sit idle) and pins it to a single P-core (`Core 1 — Performance`), dropping capacity by $-22.7\%$ (from 44 to 34 instances), with only partial recovery at 128 samples (36) and 256 samples (37).

  * **5-Track Waves L4 Buffer Inversion (`05ch @ 128 WavesL4 36.png`):** Whereas 5 tracks of Waves L4 at 32 samples run across Super Cores 11–15 (31/track), at 128 samples (`05ch @ 128 WavesL4 36.png`) **`Core 1 — Performance` is pinned near $100\%$** while **`Core 11 — Super` sits roughly half-loaded ($\sim 50\%$)** and Super Cores 12–15 run near full height. Because one track is bottlenecked by `Core 1 — Performance`, all 5 tracks are capped at 36 instances/track — matching Core 1's 1-track ceiling at 128 samples (36).

### **Scaling Regimes on the 15-Core M5 Pro (1 to 16 Channels)**

| Regime | Channels | Active Audio Cores (Observed in Capture vs. Inferred) | Echo / Trk (Total) @ 32 | Echo / Trk (Total) @ 128 | Waves L4 / Trk (Total) @ 32 | Observed / Inferred Behavior | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| **1. Super Cores Only (1:1)** | **1 ch** | **Observed:** Echo on Core 11 ($\sim 80\%$) + Core 12; L4 rotates across 11–15 | 90 (90) | 97 (97) | 44 (44) | Single-track Super Core load. | 
|  | **2 ch** | **Observed (L4):** Super Cores 11–15 | 87 (174) | 91 (182) | 37 (74) | Both tracks stay on Super Cores (no heavy P-core load). | 
|  | **3 ch** | *Inferred from per-track numbers (no capture):* 3 S-Cores | 86 (258) | — | 35 (105) | Interpolated between 2ch and 5ch captures. | 
|  | **4 ch** | *Inferred from per-track numbers (no capture):* 4 S-Cores | 86 (344) | — | 32 (128) | Interpolated between 2ch and 5ch captures. | 
|  | **5 ch** | **Observed:** All 5 Super Cores (11–15) | **84 (420)** | 90 (450) | **31 (155)** | All 5 Super Cores loaded; no heavy load on P-cores 1–10. | 
| **2. P-Core Spill (1:1)** | **6 ch** | *Inferred from per-track numbers (no capture):* 5 S + 1 P | **81** (486) | — | **29** (174) | First spill to P-cores; P-core sets limit. | 
|  | **7 ch** | *Inferred from per-track numbers (no capture):* 5 S + 2 P | — | — | **29** (203) | Flat P-Core Plateau (29 L4/trk). | 
|  | **8 ch** | *Inferred from per-track numbers (no capture):* 5 S + 3 P | **81** (648) | — | **29** (232) | Flat P-Core Plateau (81 Echo / 29 L4). | 
|  | **9 ch** | **Observed:** 9 Cores (P-Cores 6–9 + S-Cores 11–15) | 77 (693) | — | 27 (243) | 9 cores active; Core 10 idle; Cores 1–5 background. | 
|  | **10 ch** | **Observed:** 10 Cores (P-Cores 6–10 + S-Cores 11–15) | **75 (750 PEAK)** | **74 (740)** | **26 (260 PEAK)** | **Max 1:1 Saturation (10 active cores; Cores 1–5 background).** | 
| **3. Cliff #1 (11–15 ch)** | **11 ch** | **Observed:** 10 Cores (Cores 6–15 active; 1–5 background) | **45 (495)** | **47 (517)** | **19 (209)** | **10-Core Cliff:** 11 tracks share 10 cores (P-cores $\sim 70\%$ on Echo). | 
|  | **12 ch** | **Observed:** 10 Cores (Echo: 1–5 + 11–15; L4: 6–15) | 45 (540) | — | 18 (216) | 12 tracks share 10 cores; P-groups 1–5 and 6–10 interchangeable. | 
|  | **15 ch** | **Observed:** 10 Cores (Echo: 6–15, P $\sim 60\%$; L4: 1–5 + 11–15, P $\sim 75\text{–}85\%$) | 43 (645) | — | 17 (255) | 15 tracks on 10 cores: L4 ties 10ch peak ($255$ vs. $260$); Echo $-14\%$ below peak. | 
| **4. 16 Channels** | **16 ch** | **Observed (L4):** 10 Cores (Cores 1–5 $\sim 90\%$ + 11–15); *Echo inferred* | **41 (656)** | — | **14 (224)** | **L4 Cliff #2** ($-17.6\%\text{/trk}$, $-12.2\%\text{ total}$); **Echo** steps down $-4.7\%\text{/trk}$ ($+1.7\%\text{ total}$). | 

### **Cross-System Comparison Tables (32 Samples Unless Noted)**

All figures are measured with a symmetric plugin load across tracks (n/t = not tested).

#### **Waves L4**

| Tracks | M6 12-core (measured) | M5 Pro 15-core (measured) | Intel 270K Plus 8P (measured) | 
| ----- | ----- | ----- | ----- | 
| **1** | 52 | **44** | 36 (Babyface @ 48); PCIe @ 32 n/t | 
| **2** | 37/track (74) | **37/track (74)** | n/t | 
| **5** | 35/track (175) | **31/track (155)** | n/t | 
| **6** | 35/track (210) | **29/track (174)** | n/t | 
| **8** | n/t | **29/track (232)** | **36/track (288, PCIe @ 32)**; 30/track (240, Babyface @ 48) | 
| **10 (M5 Pro 1:1 Peak)** | n/t | **26/track (260 PEAK)** | n/t | 
| **12** | 17/track (204) | **18/track (216)** | n/t | 
| **15** | n/t | **17/track (255 — ties 10ch peak within resolution)** | n/t | 
| **16** | n/t | **14/track (224)** | n/t | 

#### **Ableton Echo**

| Tracks | M6 12-core (measured) | M5 Pro 15-core (measured) | Intel 270K Plus 8P (measured) | 
| ----- | ----- | ----- | ----- | 
| **1** | 102 | **90** | 89 (PCIe @ 32); 82 (Babyface @ 48) | 
| **5** | 83/track (415) | **84/track (420)** | n/t | 
| **6** | 85/track (510) *(83/trk = 498 earlier)* | **81/track (486)** | n/t; est. $\sim 87\text{–}89\text{/track}$ ($\sim 520\text{–}535$) on PCIe | 
| **8** | n/t | **81/track (648)** | **87/track (696, PCIe @ 32)**; 80/track (640, Babyface @ 48) | 
| **10 (M5 Pro 1:1 Peak)** | n/t | **75/track (750 PEAK)** | n/t | 
| **12** | 42/track (504) | **45/track (540)** | n/t | 
| **15** | n/t | **43/track (645)** | n/t | 
| **16** | n/t | **41/track (656)** | **43/track (688, PCIe @ 32)**; **41/track (656, Babyface @ 48)** | 

### **Notes on the Comparison**

* **Interfaces:** The Mac columns use the Babyface (USB) at 32 samples. Intel's Babyface @ 48 is the same-interface comparison, but at a larger buffer. Intel's PCIe @ 32 matches the buffer but not the interface.

* **M6 6-track Echo:** The CSV catalog records 85/track (510) at 32 samples, while the earlier summary table used 83/track (498); both are noted above for consistency.

* **Why 15 and 16 tracks differ between Echo and Waves L4 on M5 Pro:** In all 11–16 track tests on the 15-core M5 Pro, only 10 cores carry heavy audio load while 5 P-cores remain unused by audio. At 15 tracks, 5 cores process 2 tracks each while 5 cores process 1 track each: **Echo reaches 645 total** ($-14.0\%$ below its 10-track peak of 750), while **Waves L4 reaches 255 total**, essentially tying its 10-track peak of 260 within the $\pm 1\text{ instance/track}$ ($\pm 15\text{ total}$) test resolution. At 16 tracks, a 6th core (at least one mid-tier P-core) must take a 2nd track: **Waves L4 drops by $-12.2\%$ in total capacity to 224 (14/track)**, whereas **Echo steps down from 43 to 41/track (656 total)** — tying the 8-core 270K Plus on Babyface @ 48 (656) and trailing it on PCIe @ 32 (688).

## **Mac mini M5 Pro Activity Monitor Captures**

macOS Activity Monitor numbers the M5 Pro inversely compared to the M6 (M5 Pro = Cores 1–10 Performance, Cores 11–15 Super; M6 = Cores 1–6 Efficiency, Cores 7–8 Super, Cores 9–12 Performance). In the captures below, load patterns divide into three 5-core groups: **Cores 1–5 (Performance)**, **Cores 6–10 (Performance)**, and **Cores 11–15 (Super)**.

*(Disclaimer: **The CSV table is the authoritative record of final dropout-free maximums.** Where a screenshot filename below shows a slightly lower count — specifically `09ch Echo 73` vs. CSV `77`, and `15ch Echo 42` vs. CSV `43` — the capture was taken slightly before the instance count was raised to its final limit.)*

### 01ch macOS MacMini M5Pro Babyface@32 Echo 90

Refer to attached image: `01ch macOS MacMini M5Pro Babyface@32 Echo 90.png`

### 01ch macOS MacMini M5Pro Babyface@32 WavesL4 44

Refer to attached image: `01ch macOS MacMini M5Pro Babyface@32 WavesL4 44.png`

### 01ch macOS MacMini M5Pro Babyface@64 WavesL4 34

Refer to attached image: `01ch macOS MacMini M5Pro Babyface@64 WavesL4 34.png`

### 02ch macOS MacMini M5Pro Babyface@32 WavesL4 37

Refer to attached image: `02ch macOS MacMini M5Pro Babyface@32 WavesL4 37.png`

### 05ch macOS MacMini M5Pro Babyface@32 Echo 84

Refer to attached image: `05ch macOS MacMini M5Pro Babyface@32 Echo 84.png`

### 05ch macOS MacMini M5Pro Babyface@32 WavesL4 31

Refer to attached image: `05ch macOS MacMini M5Pro Babyface@32 WavesL4 31.png`

### 05ch macOS MacMini M5Pro Babyface@128 WavesL4 36

Refer to attached image: `05ch macOS MacMini M5Pro Babyface@128 WavesL4 36.png`

### 09ch macOS MacMini M5Pro Babyface@32 Echo 73

Refer to attached image: `09ch macOS MacMini M5Pro Babyface@32 Echo 73.png`

### 09ch macOS MacMini M5Pro Babyface@32 WavesL4 27

Refer to attached image: `09ch macOS MacMini M5Pro Babyface@32 WavesL4 27.png`

### 10ch macOS MacMini M5Pro Babyface@32 Echo 75

Refer to attached image: `10ch macOS MacMini M5Pro Babyface@32 Echo 75.png`

### 10ch macOS MacMini M5Pro Babyface@32 WavesL4 26

Refer to attached image: `10ch macOS MacMini M5Pro Babyface@32 WavesL4 26.png`

### 11ch macOS MacMini M5Pro Babyface@32 Echo 45

Refer to attached image: `11ch macOS MacMini M5Pro Babyface@32 Echo 45.png`

### 11ch macOS MacMini M5Pro Babyface@32 WavesL4 19

Refer to attached image: `11ch macOS MacMini M5Pro Babyface@32 WavesL4 19.png`

### 12ch macOS MacMini M5Pro Babyface@32 WavesL4 18

Refer to attached image: `12ch macOS MacMini M5Pro Babyface@32 WavesL4 18.png`

### 12ch macOS MacMini M5Pro Babyface@32 Echo 45

Refer to attached image: `12ch macOS MacMini M5Pro Babyface@32 Echo 45.png`

### 15ch macOS MacMini M5Pro Babyface@32 Echo 42

Refer to attached image: `15ch macOS MacMini M5Pro Babyface@32 Echo 42.png`

### 15ch macOS MacMini M5Pro Babyface@32 WavesL4 17

Refer to attached image: `15ch macOS MacMini M5Pro Babyface@32 WavesL4 17.png`

### 16ch macOS MacMini M5Pro Babyface@32 WavesL4 14

Refer to attached image: `16ch macOS MacMini M5Pro Babyface@32 WavesL4 14.png`