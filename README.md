#  Discrete-Time Internet Worm Propagation Simulation
###  Programming Project 3 | Mara Burnside | UCF | April 2026

---

## 📋 Overview

A discrete-time simulation of internet worm propagation implemented in Python. Two scanning strategies are modeled and compared across a network of 100,000 IP addresses containing 1,000 vulnerable machines. The simulation tracks infected host growth I(t) at each time tick until all vulnerable machines are compromised.

**Key finding: Local-preference scanning propagates ~26% faster than purely random scanning (508 vs. 677 ticks on average).**

---

## 📊 Results at a Glance

| Metric | Value |
|---|---|
| Total IP Space (X) | 100,000 |
| Vulnerable Machines (N) | 1,000 |
| Scans Per Tick (M) | 3 |
| Infection Activation Delay | 45 ticks |
| Initial Infected Host | IP 2010 |
| Simulation Runs Per Mode | 4 |

---

## 🔬 Part 1 — Random Scanning

### How It Works

The script `worm_sim.py` simulates a worm that scans entirely at random. At each time tick, every active infected host performs 3 scans using `random.randint(1, X)`. If a scanned IP belongs to a vulnerable, uninfected machine, it becomes infected — but waits 45 ticks before it begins scanning itself. Vulnerable machines are arranged into structured clusters via `generate_vulnerable()`, and the simulation starts with a single infected host at IP 2010.

### Finish Times

| Run | Finish Time (ticks) |
|---|---|
| 1 | 679 |
| 2 | 692 |
| 3 | 673 |
| **4** | **665** *(fastest)* |
| **Average** | **677** |

### Infection Growth Plot

![Random Scanning I(t)](random%20scanning%20plots.png)

> The flat early phase reflects the 45-tick activation delay. Once enough hosts become active, infection accelerates exponentially before saturating as the vulnerable pool depletes — a classic S-curve epidemic pattern.

---

## 🎯 Part 2 — Local-Preference Scanning

### How It Works

The script `worm_local_preference.py` implements a biased scanning strategy. The core behavior is handled by `local_preference_scan()`:

- **80% of the time** — scan within ±10 IPs of the infected host (local range)
- **20% of the time** — scan a completely random IP across the full address space

All other parameters (X, N, M, delay, initial host) remain identical to Part 1, making it a direct controlled comparison.

```python
def local_preference_scan(infected_ip, X, local_range=10):
    if random.random() < 0.8:
        low  = max(1, infected_ip - local_range)
        high = min(X, infected_ip + local_range)
        return random.randint(low, high)
    else:
        return random.randint(1, X)
```

### Finish Times

| Run | Finish Time (ticks) |
|---|---|
| **3** | **479** *(fastest)* |
| 1 | 495 |
| 2 | 498 |
| 4 | 560 *(outlier)* |
| **Average** | **508** |

> Run 4's slower finish time (560 ticks) is likely due to the initial cluster being more isolated — a known stochastic effect of local scanning when nearby vulnerable machines are sparse.

### Infection Growth Plot

![Local Preference I(t)](local%20preference%20plot.png)

> Faster overall propagation compared to random scanning. The higher run-to-run variance (479–560, an 81-tick spread vs. only 27 for random scanning) shows that topology luck plays a larger role when scanning locally.

---

## 📈 Comparison Summary

| Metric | Random Scanning | Local-Preference |
|---|---|---|
| Average Finish Time | 677 ticks | **508 ticks** |
| Fastest Run | 665 ticks | **479 ticks** |
| Slowest Run | 692 ticks | 560 ticks |
| Run-to-Run Spread | 27 ticks | 81 ticks |
| Speed Advantage | — | **~26% faster** |

---

## 🗂️ Repository Structure

```
project/
├── worm_sim.py                    # Part 1 — random scanning simulation
├── worm_local_preference.py       # Part 2 — local-preference scanning simulation
├── random scanning plots.png      # I(t) plot for Part 1 (4 runs)
├── local preference plot.png      # I(t) plot for Part 2 (4 runs)
├── RESULTS.xlsx                   # Raw I(t) data (two sheets: Random, Local)
├── random_results.csv             # CSV output from worm_sim.py
├── local_preference_results.csv   # CSV output from worm_local_preference.py
└── README.md
```

---

## ▶️ Usage

```bash
# Part 1 — Random Scanning
python3 worm_sim.py

# Part 2 — Local-Preference Scanning
python3 worm_local_preference.py
```

Both scripts run 4 independent simulations, print progress to the terminal, display a formatted finish-time table, and save I(t) data to a CSV file.

**Expected terminal output (Part 2 example):**
```
Run 1 complete — all 1000 machines infected at tick 495
Run 2 complete — all 1000 machines infected at tick 498
Run 3 complete — all 1000 machines infected at tick 479
Run 4 complete — all 1000 machines infected at tick 560

Finish Times:
  Run 1: 495
  Run 2: 498
  Run 3: 479
  Run 4: 560
```

---

## 🔍 Key Findings

**Local preference is ~26% faster** — Average finish of 508 ticks vs. 677 for random scanning. Exploiting clustered topology significantly accelerates propagation, with direct implications for network segmentation as a defense.

**S-curve epidemic pattern** — Both models follow classic logistic growth: a slow initial phase (45-tick delay), exponential spread as active hosts multiply, then saturation as vulnerable machines run out.

**Higher variance in local mode** — Local-preference runs spanned 81 ticks (479–560) vs. just 27 for random scanning. Topology luck matters more when scanning locally — an infected host near a sparse cluster will spread slowly regardless of strategy.

**Infection delay shapes early spread** — The mandatory 45-tick activation delay creates a flat phase in all plots. Lowering this parameter would dramatically steepen the early growth curve.

**Cluster topology is exploitable** — The `generate_vulnerable()` clustering means local scanning hits dense vulnerable regions efficiently. Randomized IP allocation would reduce this attack surface in real deployments.

---

## 🖥️ Environment

- **Platform:** UCF Eustis Linux machine
- **Language:** Python 3
- **Course:** CAP 6135 – Malware and Software Vulnerability Analysis
- **Execution:** `python3 worm_sim.py` / `python3 worm_local_preference.py`

---

*CAP 6135 · Programming Project 3 · University of Central Florida · April 2026*
