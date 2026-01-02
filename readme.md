The Elastic Engine (v6.0)
An open research proposal for exploiting crankshaft torsional dynamics via Active Phase Alignment.

Note: This project is currently in the Pre-Validation Proposal phase. We are actively seeking peer review, contributors for simulation modeling, and critical analysis of our control architecture.

📖 Table of Contents
Executive Summary

The Physics

System Architecture

Technology Stack

Development Roadmap

Safety & Compliance

Contributing

⚡ Executive Summary
Contemporary Internal Combustion Engine (ICE) control treats the crankshaft as a rigid power transmission element. In reality, under high load, crankshafts exhibit torsional compliance between 0.1° and 2.0°. Current engineering suppresses this strain energy via heavy mass dampers, dissipating it as heat.



The Elastic Engine hypothesis is that this strain energy can be recovered as useful shaft work by timing combustion events to apply torque in-phase with the elastic rebound velocity.


Target Efficiency Gain: 0.2%–0.5% BSFC improvement.


Power Recovery: 50–200 Watts continuous; up to 2 kW at resonance peaks.


Secondary Benefit: Potential removal of harmonic damper mass (1–3 kg) in Phase 2.

🧪 The Physics
Torque-Velocity Phase Alignment
The core mechanism is Active Phase Alignment. Instead of fighting torsional vibration, we exploit it.


Strain Phase: Combustion applies torque, twisting the crank and storing potential energy (U=½Kθ 
2
 ).


Rebound Phase: As pressure drops, the crank "unwinds".


Recovery: We time the next combustion event to apply pressure while the crank is rebounding (ω 
rebound
​
 >0), maximizing instantaneous power transfer (P=τ×ω).

Mathematical Formulation
To achieve net positive work, we apply a timing offset Δϕ such that:

∫[τ(t)×ω 
torsional
​
 (t)]dt>0
The control law applied is: Δφ = G × (θ_measured − θ_predicted) × sign(ω_torsional)


Where G is a gain factor bounded by authority limits (±3° CA). 

🏗 System Architecture
The system utilizes a Dual-Layer Supervisor topology to separate optimization from safety-critical operations.

Layer 1: The Deterministic ECU (Safety Net)

Platform: AUTOSAR Classic 4.4 on Infineon AURIX TC3xx.

Role: Standard engine control. Monitors Layer 2 heartbeat.


Fail-Safe: If Layer 2 fails, engine reverts to standard maps indefinitely.

Layer 2: The FPGA Torsional Supervisor

Platform: Xilinx Zynq UltraScale+ (ZCU106).


Role: High-speed torsional state estimation and injection timing offset calculation.

Latency: Hard real-time requirement. End-to-End latency budget is 108–128 µs.

Component	Latency (µs)
Sensor Acquisition	10
ADC Conversion	2
FPGA Signal Processing	5
Prediction Algorithm (MPC)	8
Injector Response	80–100
Total	~108–128


Note: Injection timing relies on hardware interrupts from the crankshaft encoder, NOT CAN bus messages. 

💻 Technology Stack

Hardware: Xilinx Zynq UltraScale+ (ARM Cortex-A53 + FPGA fabric).


Sensors: MagCanica MTS-100HT (Primary), Methode TS-200 (Secondary).


Algorithms:


MPC: Linear-parameter-varying (LPV) model with 4 states.


State Estimation: Extended Kalman Filter (EKF) for transient convergence.


Fatigue Monitoring: Rainflow cycle counting (ASTM E1049-85) + Miner's Rule.

🛣 Development Roadmap
We are currently initiating Phase 1.

Phase 1: Subsystem Characterisation (Months 0–14) 🟡 Current Status

Goal: Validate sensor thermal survival (160°C) and prediction algorithms.

Gate Criteria: >99% prediction confidence, <0.5% sensor drift.

Phase 2: Active Damping Validation (Months 14–24) 🔴

Goal: Demonstrate 50% torsional amplitude reduction without physical damper.

Phase 3: Efficiency Exploitation (Months 24–36) 🔴

Goal: Demonstrate >0.2% BSFC improvement.

🛡 Safety & Compliance
This project adheres to strict safety protocols regarding "Start of Injection" authority.

Hardware Limits: An analog comparator on the FPGA board physically clips timing offsets to ±3° CA. This is tamper-sealed.


EU AI Act: The system uses deterministic Model Predictive Control (MPC), not neural networks. It is currently classified outside the scope of "AI Systems" under EU Regulation 2024/1689.



ISO 26262: Fatigue monitoring is ASIL-C compliant.

🤝 Contributing
We are looking for contributors to help validating V6.0 of the proposal.

Areas for Contribution

Control Theory: Reviewing the LPV model structure and EKF convergence rates (referencing Quartullo et al., 2023).


FPGA/RTL: Optimizing the solver implementation for the Zynq architecture to ensure <8 µs solve times.


Simulation: Building independent MATLAB/Simulink models to Red Team the "Efficiency Ceiling" claims.

How to Submit
Check the Issues tab for open "Request for Comment" (RFC) threads.

Fork the repo and propose changes to the docs/ architecture folders.

All PRs regarding the fatigue model must cite ASTM E1049-85 compliance.

Legal & Liability
Licensing: [Insert License Here - e.g., MIT, Apache 2.0, or Proprietary].

Liability: Per UCTA 1977, liability for death/personal injury cannot be excluded. All contributors must acknowledge that code provided here is for research purposes only and not for deployment on public roads without certification.


Based on "The Elastic Engine V6.0: Feasibility Study & Control Architecture" - Jan 2026