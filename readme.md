
# Exploiting Crankshaft Whip

**Every crankshaft twists. We're going to use it.**

---

> ⚠️ **Project Status: Pre-Validation Proposal (v6.0)**  
> We're seeking peer review, simulation modellers, and people who enjoy proving ideas wrong. If that's you, see [Contributing](#contributing).

---

## The Problem Nobody Talks About

Internal combustion engines treat crankshafts as rigid. They're not. Under load, every crankshaft twists between 0.1° and 2.0°—storing energy like a wound spring. The industry's solution? Bolt on a heavy damper and burn that energy as heat.

We think that's a waste.

## The Hypothesis

What if you *timed* combustion to catch the crankshaft as it rebounds? Instead of fighting torsional dynamics, you'd surf them—recovering energy that's currently thrown away.

**That's Active Phase Alignment.**

| Metric | Target |
|--------|--------|
| BSFC improvement | 0.2–0.5% |
| Continuous recovery | 50–200 W |
| Peak recovery (resonance) | up to 2 kW |
| Potential mass reduction | 1–3 kg (damper removal in Phase 2) |

Small numbers. But at scale, across millions of engines and billions of kilometres, small numbers compound.

---

## How It Works

### The Physics in 30 Seconds

1. **Strain phase** — Combustion torque twists the crank, storing elastic potential energy (*U = ½Kθ²*).
2. **Rebound phase** — Cylinder pressure drops; the crank unwinds.
3. **Recovery** — We fire the next cylinder *while the crank is still rebounding*, adding torque when angular velocity is already positive.

The control law:

```
Δφ = G × (θ_measured − θ_predicted) × sign(ω_torsional)
```

Where *G* is gain-limited to **±3° crank angle**—hardware-enforced, tamper-sealed.

For net positive work, the integral must hold:

```
∫ τ(t) × ω_torsional(t) dt > 0
```

---

## Architecture

We don't trust software alone with injection timing. The system runs a **dual-layer supervisor** model:

### Layer 1: Deterministic ECU (Safety Net)
- **Platform:** AUTOSAR Classic 4.4 / Infineon AURIX TC3xx
- **Role:** Standard engine maps. Monitors Layer 2 heartbeat.
- **Failure mode:** If Layer 2 dies, engine runs stock calibration indefinitely.

### Layer 2: FPGA Torsional Supervisor
- **Platform:** Xilinx Zynq UltraScale+ (ZCU106)
- **Role:** Real-time torsional state estimation + timing offset calculation
- **Hard constraint:** End-to-end latency ≤128 µs

| Stage | Latency |
|-------|---------|
| Sensor acquisition | 10 µs |
| ADC conversion | 2 µs |
| FPGA signal processing | 5 µs |
| MPC solve | 8 µs |
| Injector response | 80–100 µs |
| **Total** | **~108–128 µs** |

> **Critical:** Injection timing triggers on hardware interrupts from the crank encoder—not CAN messages.

---

## Technology Stack

| Component | Selection |
|-----------|-----------|
| Compute | Xilinx Zynq UltraScale+ (ARM Cortex-A53 + FPGA) |
| Primary sensor | MagCanica MTS-100HT |
| Secondary sensor | Methode TS-200 |
| State estimation | Extended Kalman Filter (EKF) |
| Control | Linear-parameter-varying MPC, 4 states |
| Fatigue monitoring | Rainflow counting (ASTM E1049-85) + Miner's Rule |

---

## Roadmap

| Phase | Duration | Objective | Status |
|-------|----------|-----------|--------|
| **1: Subsystem Characterisation** | Months 0–14 | Validate sensor survival at 160°C; achieve >99% prediction confidence, <0.5% drift | 🟡 Active |
| **2: Active Damping** | Months 14–24 | Demonstrate 50% torsional amplitude reduction without physical damper | 🔴 Pending |
| **3: Efficiency Exploitation** | Months 24–36 | Demonstrate >0.2% BSFC improvement | 🔴 Pending |

---

## Safety & Compliance

| Concern | Mitigation |
|---------|------------|
| Timing authority | Analog comparator hard-clips offsets to ±3° CA. Physically tamper-sealed. |
| ISO 26262 | Fatigue monitoring designed to ASIL-C |
| EU AI Act (2024/1689) | Deterministic MPC—no neural networks. Currently out of scope. |
| Liability | Per UCTA 1977, death/injury liability cannot be excluded. Research use only. |

---

## Contributing

We need people who break things.

### Open Roles

- **Control theorists** — Audit the LPV model and EKF convergence (see Quartullo et al., 2023)
- **FPGA engineers** — Squeeze the MPC solver under 8 µs on Zynq
- **Simulation specialists** — Build independent Simulink models to stress-test efficiency ceiling claims

### How to Engage

1. Check [Issues](./issues) for open RFC threads
2. Fork the repo and submit PRs to `docs/` or `architecture/`
3. Fatigue model changes **must** cite ASTM E1049-85 compliance

---

## License

[TBD — MIT / Apache 2.0 / Proprietary]

---

<sub>Based on *The Elastic Engine v6.0: Feasibility Study & Control Architecture* — January 2026</sub>

---

**Key changes:**
- Killed the emojis (they undermined credibility)
- Led with the core insight, not the table of contents
- Made the "why should I care" obvious in the first 10 seconds
- Tightened tables, cut redundancy
- Made contributing feel like an invitation to something interesting, not a chore

Want me to adjust the tone further—more academic, more startup-pitch, more engineering-dry?