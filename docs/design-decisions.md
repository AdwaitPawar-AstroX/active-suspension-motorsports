# 🔩 Design Decisions

This document explains the key engineering decisions made during the development of the F1-inspired semi-active suspension system. Each decision is documented with the problem it addresses, the options considered, and the reasoning behind the final choice.

---

## 1. Suspension Type — Why Double Wishbone Pushrod?

### Problem
Selecting a suspension geometry that provides precise wheel control, low centre of gravity, and compatibility with electronic damping actuation under high-speed motorsports conditions.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| MacPherson Strut | Simple, lightweight, low cost | Poor camber control, limited tuning |
| Multi-link | Excellent kinematics | Complex, heavy, difficult to package |
| Double Wishbone | Precise camber control, tuneable | More complex than MacPherson |
| **Double Wishbone + Pushrod (chosen)** | All wishbone benefits + inboard damper mounting, low CG | Most complex to design |

### Decision
**F1-style double wishbone with pushrod and rocker** was selected because:
- Pushrod transfers wheel motion inboard, allowing damper to be mounted horizontally within the chassis
- Horizontal damper placement significantly lowers the centre of gravity
- Rocker arm creates a mechanical advantage point — ideal for integrating the micro linear actuator
- Mirrors proven Formula 1 suspension philosophy at scale

---

## 2. Operation Mode — Why Semi-Active over Fully Active?

### Problem
Choosing between passive, semi-active, and fully active suspension operation modes.

### Options Considered

| Mode | Description | Pros | Cons |
|---|---|---|---|
| Passive | Fixed spring-damper, no control | Simple, reliable, low cost | Cannot adapt to terrain |
| **Semi-Active (chosen)** | Variable damping, controlled electronically | Responsive, power efficient, controllable | Requires sensors and MCU |
| Fully Active | Full force generation via hydraulics/pneumatics | Maximum control authority | Very high power consumption, complex, heavy |

### Decision
**Semi-active** was selected because:
- Provides meaningful performance improvement over passive systems without the power and weight penalty of fully active
- Micro linear actuators (4–6W) are sufficient to modulate damping stiffness in real time
- Response time of < 10ms meets motorsports requirements
- Practical to prototype and validate within project scope

---

## 3. Predictive Control — Why Add LIDAR?

### Problem
Traditional reactive suspension systems respond to terrain disturbances only after wheel contact — introducing unavoidable lag between disturbance and correction.

### Options Considered

| Approach | Description | Limitation |
|---|---|---|
| Reactive only (PID/LQR) | Responds to sensor feedback after contact | Always has response lag |
| Camera-based prediction | Uses vision to predict terrain | Computationally heavy, lighting dependent |
| **LIDAR predictive (chosen)** | Maps terrain 1.5–2m ahead, pre-adjusts suspension | Higher cost, requires fusion with reactive path |

### Decision
**Solid-state LIDAR** was selected for predictive terrain mapping because:
- Provides accurate distance measurements independent of lighting conditions
- 1.5–2m scan range gives sufficient time for suspension pre-loading before wheel contact
- Enables pre-emptive damping adjustment — eliminating the reactive lag problem fundamentally
- Pairs naturally with a Kalman filter for reliable sensor fusion with the reactive path

---

## 4. Control Strategy — Why Both PID and LQR?

### Problem
Selecting the optimal control algorithm for real-time damping modulation.

### Options Considered

| Controller | Approach | Pros | Cons |
|---|---|---|---|
| **PID (implemented first)** | Classical feedback, error-based | Simple, well-understood, easy to tune | Not optimal, single-variable focus |
| **LQR (planned)** | State-space optimal control | Minimises cost function across all states | Requires accurate system model |
| MPC (Model Predictive Control) | Optimises over future horizon | Most powerful | Computationally expensive for real-time |

### Decision
**Both PID and LQR** are being implemented and compared:
- PID serves as the baseline — simpler to implement and validate first
- LQR is expected to outperform PID by simultaneously managing body displacement, velocity, and actuator effort as a combined cost function
- Comparing both controllers in simulation provides quantitative evidence of performance difference — strengthening the research value of the project
- MPC was considered but deemed too computationally demanding for the current MCU hardware

---

## 5. Sensor Fusion — Why Kalman Filter?

### Problem
The predictive (LIDAR) and reactive (IMU + actuator) data streams need to be intelligently combined. Naively applying both simultaneously could cause conflicting actuator commands.

### Options Considered

| Approach | Description | Limitation |
|---|---|---|
| Simple switching | Use LIDAR when available, reactive otherwise | Abrupt transitions, unstable |
| Weighted average | Fixed weighting of both signals | Doesn't account for signal reliability |
| **Kalman Filter (chosen)** | Probabilistic fusion based on signal confidence | More complex but reliable and smooth |

### Decision
**Kalman filter** was selected for sensor fusion because:
- Dynamically weights LIDAR vs reactive feedback based on measurement confidence at each timestep
- Produces smooth, stable control transitions between predictive and reactive modes
- Industry-standard approach for sensor fusion in automotive and robotics applications
- Directly applicable to future autonomous vehicle research

---

## 6. Steering System — Why Dual Independent Servos?

### Problem
Selecting a steering mechanism that replicates realistic F1-grade geometry and cornering dynamics.

### Options Considered

| Option | Description | Limitation |
|---|---|---|
| Single servo | One servo controls both wheels via linkage | Cannot achieve true Ackermann correction |
| Rack and pinion | Mechanical linkage, traditional | Fixed geometry, no electronic correction |
| **Dual independent servo (chosen)** | Two servos, MCU-synchronized | Slightly more complex, requires synchronisation logic |

### Decision
**Dual-servo independent steering** was selected because:
- Left and right steering arms can be controlled at different angles simultaneously
- Enables true Ackermann geometry correction — inner wheel turns tighter than outer wheel during cornering
- MCU synchronisation ensures both servos respond cohesively
- Produces significantly tighter cornering radius and reduces understeer compared to single-servo solutions
- Directly mirrors F1 front suspension steering principles

---

## 7. Material Selection — Phased Approach

### Problem
Selecting materials that balance structural strength, weight, machinability, and cost — while allowing rapid design iteration during early prototyping before committing to final production materials.

### Phased Material Strategy

A two-phase material approach was adopted deliberately:

---

#### Phase 1 — Prototype (Current)
**All structural components 3D printed in PETG-CF (Carbon Fibre reinforced PETG)**

| Component | Material | Reasoning |
|---|---|---|
| Wishbone arms | PETG-CF | Stiff, lightweight, printable, allows rapid design iteration |
| Rocker arms | PETG-CF | Complex geometry easily achieved with 3D printing |
| Chassis structure | PETG-CF | Quick to modify and reprint if design changes needed |
| Brackets & mounts | PETG-CF | Fast iteration without CNC lead time or cost |

**Why PETG-CF for prototyping:**
- Higher stiffness than standard PETG due to carbon fibre reinforcement
- Good layer adhesion and dimensional stability under mechanical load
- Allows full geometry validation before investing in expensive final materials
- Design changes cost only filament and print time — not machining hours
- Sufficient strength for low-load functional testing and fit checks

---

#### Phase 2 — Production (Planned)
**Switch to final motorsports-grade materials once design is validated**

| Component | Prototype Material | Final Material | Reason for Switch |
|---|---|---|---|
| Wishbone arms | PETG-CF | CF-Nylon / Carbon Fibre composite | Higher strength-to-weight, fatigue resistance |
| Rocker arms | PETG-CF | CNC Aluminium | Precision geometry, higher load capacity |
| Coilover spring-damper | PETG-CF (housing) | Steel | Fatigue resistance under dynamic cycling |
| Chassis | PETG-CF | Aluminium / CF composite | Stiffness and durability under real loads |

**Transition criteria:** Phase 2 materials will be adopted once:
- Geometry is fully validated through prototype testing
- Control system is tuned and verified in simulation
- Load analysis confirms exact force requirements per component

---

### Why This Approach is Sound Engineering Practice

Phased material prototyping is standard in motorsports and aerospace engineering:
- Reduces cost and time during iterative design phase
- Prevents expensive rework of machined or composite parts due to design changes
- Allows focus on control system development in parallel with mechanical refinement
- Mirrors professional engineering workflows used in F1 development cycles

---

## 8. Open Design Questions (Ongoing R&D)

These decisions have not yet been finalised and are under active investigation:

### Actuator Configuration
**Question:** Should the micro linear actuator be mounted in parallel with the physical damper, or should it replace the physical damper entirely?

| Option | Pros | Cons |
|---|---|---|
| **Parallel (Preferred)** | Mechanical damper handles base load, actuator modulates only | Slightly heavier, more complex packaging |
| **Replacement** | Simpler packaging, fewer components | High power consumption, heat generation, reliability risk |

> Current direction: Parallel configuration preferred. Further simulation needed to confirm actuator force requirements.

---

## 📝 Notes

- All design decisions are subject to revision as simulation and prototype testing provide real-world data
- This document will be updated as new decisions are made and previous ones are validated or revised
- Engineering decisions prioritise: performance → reliability → weight → cost

---

*Last updated: March 2026*
