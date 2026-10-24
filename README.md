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

* **Apple Silicon (New):** Mac mini M5 Pro (15-Core: 5 Super Cores, 10 mid-tier Performance Cores, 0 Efficiency Cores; 24 GB Unified Memory) running macOS Golden Gate. Note that Activity Monitor numbers the M5 Pro inversely compared to the M6: **Cores 1–10 are Performance cores**, and **Cores 11–15 are Super Cores**. Across all captures, load patterns divide cleanly into three 5-core groups: **Cores 1–5 (P-group A)**, **Cores 6–10 (P-group B)**, and **Cores 11–15 (Super Cores)**.

* **Apple Silicon (Reference from Part 1):** Mac mini M6 (12-Core: Cores 1–6 Efficiency, Cores 7–8 Super, Cores 9–12 Performance; 24 GB) on macOS Golden Gate.

* **x86 Reference (from Part 1):** Intel Core Ultra 7 270K Plus (8 Lion Cove P-cores used