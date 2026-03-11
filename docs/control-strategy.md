# 🧠 Control Strategy

This document details the control system architecture, algorithms, and implementation approach for the F1-inspired semi-active suspension system.

---

## Overview

The control system integrates three layers working simultaneously:

```
Layer 1 — Predictive Control    : LIDAR terrain scanning → pre-emptive damping
Layer 2 — Reactive Control      : IMU + sensors → real-time damping correction  
Layer 3 — Steering Control      : Dual-servo Ackermann correction
```

All three layers are managed by a central MCU running in real time with a system response target of **< 10ms**.

---

## 1. System Model — Quarter-Car Model

The suspension dynamics are modelled using a **quarter-car model** — a standard representation in vehicle dynamics research.

### State Variables

| Variable | Symbol | Description |
|---|---|---|
| Sprung mass displacement | x₁ | Vertical displacement of vehicle body |
| Sprung mass velocity | ẋ₁ | Vertical velocity of vehicle body |
| Unsprung mass displacement | x₂ | Vertical displacement of wheel assembly |
| Unsprung mass velocity | ẋ₂ | Vertical velocity of wheel assembly |
| Road input | w | Surface disturbance input |

### Equations of Motion

```
Sprung mass (body):
m₁ẍ₁ = -ks(x₁ - x₂) - cs(ẋ₁ - ẋ₂) + u

Unsprung mass (wheel):
m₂ẍ₂ = ks(x₁ - x₂) + cs(ẋ₁ - ẋ₂) - kt(x₂ - w) - u
```

Where:
- `m₁` = sprung mass (vehicle body)
- `m₂` = unsprung mass (wheel + hub assembly)
- `ks` = suspension spring stiffness
- `cs` = damping coefficient
- `kt` = tyre stiffness
- `u`  = active control force from actuator

---

## 2. PID Controller

### Concept
The PID controller generates a control force `u` based on the error between desired and actual sprung mass displacement.

### Control Law
```
u(t) = Kp·e(t) + Ki·∫e(t)dt + Kd·(de/dt)

Where:
e(t)  = desired displacement - actual displacement
Kp    = Proportional gain
Ki    = Integral gain  
Kd    = Derivative gain
```

### Tuning Approach
- **Proportional (Kp):** Controls magnitude of response — tuned first to reduce oscillation
- **Integral (Ki):** Eliminates steady-state error — tuned to remove persistent offset
- **Derivative (Kd):** Dampens overshoot — tuned to improve transient response

### Implementation Status
🔄 In Progress — Gains being tuned against MATLAB/Simulink quarter-car simulation

### Performance Metrics Being Evaluated
- Body displacement (mm) under step and sinusoidal road inputs
- Settling time (ms)
- Peak overshoot (%)
- RMS acceleration of sprung mass

---

## 3. LQR Controller (Linear Quadratic Regulator)

### Concept
LQR is an **optimal state-space controller** that minimises a cost function balancing ride comfort (body acceleration) against road handling (tyre contact force) and actuator effort (energy consumption).

### State-Space Representation
```
ẋ = Ax + Bu + Ew
y  = Cx

Where:
x = [x₁, ẋ₁, x₂, ẋ₂]ᵀ   (state vector)
u = control force (actuator output)
w = road disturbance input
```

### Cost Function
```
J = ∫₀^∞ (xᵀQx + uᵀRu) dt

Where:
Q = state weighting matrix (penalises body displacement and acceleration)
R = control weighting matrix (penalises actuator effort)
```

### Gain Computation
The optimal gain matrix **K** is computed by solving the **Algebraic Riccati Equation (ARE)**:
```
u = -Kx
K = R⁻¹BᵀP
AᵀP + PA - PBR⁻¹BᵀP + Q = 0
```

### Tuning Approach
- Q and R matrices adjusted to trade off between comfort and handling
- Higher Q weighting on body acceleration → prioritises ride comfort
- Higher R weighting → penalises actuator use, reduces energy consumption

### Implementation Status
⏳ Planned — begins after PID baseline is validated in simulation

### Expected Advantage over PID
- Simultaneously manages all four state variables as a system
- Mathematically optimal for the defined cost function
- Expected lower RMS body acceleration and better transient response than PID

---

## 4. Kalman Filter — Sensor Fusion

### Problem
LIDAR predictive data and reactive sensor data (IMU, displacement sensors) must be intelligently fused. Simply averaging them produces instability — the system needs to determine which signal is more reliable at each timestep.

### Kalman Filter Concept
The Kalman filter maintains a **probabilistic estimate** of the true system state, combining:
- **Prediction** — based on system model (what we expect to happen)
- **Measurement update** — based on sensor readings (what we actually observe)

### Two-Input Fusion Architecture
```
Input 1: LIDAR scan → predicted surface height → expected suspension demand
Input 2: IMU + sensors → actual suspension state → reactive correction demand

Kalman Filter:
→ Weights each input by its covariance (measurement uncertainty)
→ Higher confidence input gets higher weight
→ Outputs optimal fused control signal to MCU
```

### Innovation (Key Engineering Insight)
When LIDAR confidence is high (clear terrain ahead):
- Predictive signal dominates → suspension pre-loads **before** wheel contact
- Eliminates reactive lag entirely for that disturbance

When LIDAR confidence is low (sensor uncertainty, close-range obstacles):
- Reactive signal dominates → system falls back to pure PID/LQR feedback
- Ensures safety and reliability under all conditions

### Implementation Status
⏳ Planned — after LQR implementation is complete

---

## 5. Dual-Servo Steering Control

### Ackermann Geometry Correction
During cornering, the inner wheel must turn at a sharper angle than the outer wheel to prevent tyre scrub. This is the **Ackermann principle**.

### Control Law
```
For a turn of radius R and vehicle track width T:

Inner wheel angle:  δ_inner = arctan(L / (R - T/2))
Outer wheel angle:  δ_outer = arctan(L / (R + T/2))

Where L = wheelbase
```

### MCU Implementation
- Driver input → MCU calculates required turn radius R
- MCU independently commands left and right servos to δ_inner and δ_outer respectively
- Potentiometer / magnetic encoder provides real-time angle feedback
- PID correction loop maintains target angles under load

### Steering Specifications
- Servo torque: 30–35 kg·cm
- Steering ratio: ~15:1 (adjustable)
- Feedback: Real-time via potentiometer / magnetic encoder

---

## 6. Control System Integration

### Full System Loop

```
Every control cycle (target < 10ms):

1. LIDAR scans terrain ahead → predictive demand calculated
2. IMU + sensors read current state → reactive demand calculated
3. Kalman filter fuses both signals → optimal control demand
4. MCU sends command to:
   a. Damper actuators (stiffness adjustment)
   b. Steering servos (Ackermann correction)
5. Sensors confirm actuator response → loop repeats
```

### MCU Requirements
- Real-time processing capability for < 10ms loop time
- Simultaneous handling of multiple sensor inputs
- PWM output for servo and actuator control
- I2C/SPI communication with IMU and LIDAR

---

## 7. Simulation Plan

All controllers will be validated in **MATLAB/Simulink** before hardware implementation:

| Simulation | Purpose | Status |
|---|---|---|
| Quarter-car passive baseline | Establish reference performance | 🔄 In Progress |
| PID controller | Validate and tune gains | 🔄 In Progress |
| LQR controller | Compare against PID | ⏳ Planned |
| Sensor fusion (Kalman) | Validate fusion stability | ⏳ Planned |
| Full predictive-reactive loop | End-to-end system validation | ⏳ Planned |

### Key Simulation Inputs
- Step input — sudden bump
- Sinusoidal input — continuous road undulation
- Random road profile — realistic motorsports surface

### Performance Metrics
- RMS sprung mass acceleration (ride comfort)
- Tyre deflection (road handling)
- Actuator force demand (energy efficiency)
- Settling time and overshoot

---

> 📝 This document will be updated as simulation results and hardware testing data become available. Last updated: **March 2026**
