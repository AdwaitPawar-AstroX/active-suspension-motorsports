# 🔍 System Overview

A simplified, accessible explanation of the complete suspension system — what it does, why it matters, and how all the parts work together.

---

## What Problem Does This Solve?

In motorsports, a vehicle travelling at high speed encounters constant surface irregularities — bumps, kerbs, dips, and uneven tarmac. A conventional **passive suspension** system uses fixed springs and dampers. It absorbs shocks reasonably well, but it cannot adapt — it responds the same way to a gentle curve as it does to a sharp kerb.

The result:
- Unpredictable handling at high speed
- Driver fatigue from constant vibration
- Reduced tyre contact with the road — less grip, less safety
- Inconsistent aerodynamic downforce as the car pitches and rolls

**The goal of this project is to build a suspension system that thinks ahead.**

---

## The Solution — Three Integrated Subsystems

This project integrates three subsystems into one unified control architecture:

```
┌─────────────────────────────────────────────┐
│                                             │
│   1. Semi-Active Suspension                 │
│      Adjusts damping stiffness in real time │
│                                             │
│   2. LIDAR Predictive Mechanism             │
│      Sees the road ahead and prepares       │
│                                             │
│   3. Dual-Servo Steering                    │
│      Replicates F1 cornering geometry       │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Subsystem 1 — Semi-Active Suspension

### The Mechanical Foundation
The suspension uses an **F1-style double wishbone with pushrod and rocker** geometry:

- Each wheel is connected to the chassis through upper and lower wishbone arms
- A pushrod transfers vertical wheel motion inward to a rocker arm
- The rocker arm compresses a **horizontally mounted coilover** (spring + damper) positioned inside the chassis
- This keeps the heavy damper components low and central — improving the vehicle's centre of gravity

### The Electronic Layer
Alongside each mechanical damper sits a **micro linear actuator** (4–6W). This actuator:
- Receives commands from the central MCU
- Adjusts the damper's internal flow resistance in real time
- Changes the suspension from soft (comfort) to stiff (performance) within **< 10ms**

This is what makes it **semi-active** — the spring and damper are still mechanical, but their behaviour is electronically controlled.

### Sensors Feeding the System
| Sensor | What it Measures |
|---|---|
| IMU (Inertial Measurement Unit) | Roll, pitch, yaw of the vehicle body |
| Ride-height sensors | Distance between chassis and ground at each corner |
| Displacement/pressure sensors | Compression state of each damper |

---

## Subsystem 2 — LIDAR Predictive Mechanism

### The Problem with Reactive Systems
Even the fastest reactive suspension has a fundamental limitation — it can only respond **after** the wheel hits a bump. By then, the disturbance has already entered the chassis.

### The LIDAR Solution
A **solid-state LIDAR** (Light Detection and Ranging) sensor is mounted at the front of the vehicle. It continuously scans the road surface **1.5–2 metres ahead**, detecting:
- Bumps and raised surfaces
- Dips and depressions
- Kerbs and sharp edges

This gives the MCU advance warning — allowing it to pre-adjust suspension stiffness **before** the wheel makes contact.

### How Predictive and Reactive Work Together

```
LIDAR sees bump 1.5m ahead
        ↓
MCU pre-loads suspension stiffness (predictive)
        ↓
Wheel hits bump
        ↓
IMU and sensors confirm actual impact (reactive)
        ↓
Kalman filter fuses both signals → final actuator command
        ↓
Damper responds with optimal stiffness
```

The **Kalman filter** is the intelligence layer — it continuously evaluates whether to trust the LIDAR prediction or the reactive sensor data more, based on measurement confidence. This ensures the system remains safe even when LIDAR readings are uncertain.

---

## Subsystem 3 — Dual-Servo Independent Steering

### The Ackermann Principle
When a car turns, the inner wheel traces a tighter arc than the outer wheel. For both tyres to roll cleanly without scrubbing, they must point at slightly different angles — this is called **Ackermann geometry**.

Most simple vehicles use a mechanical linkage that approximates this. Formula 1 cars calculate and apply it precisely in real time.

### The Dual-Servo Approach
This system uses **two independent servos** — one for the left steering arm, one for the right. The MCU:
1. Receives the desired steering input
2. Calculates the exact Ackermann-corrected angle for each wheel independently
3. Commands both servos simultaneously
4. Confirms actual angle via potentiometer / magnetic encoder feedback

### Result
- Tighter cornering radius
- Reduced tyre scrub and understeer
- More realistic F1-grade front-end response

---

## How It All Works Together

```
Vehicle travelling at speed →

LIDAR scans 1.5m ahead           IMU reads current body motion
        ↓                                    ↓
   Predictive demand              Reactive demand
        ↓                                    ↓
        └──────────┬─────────────────────────┘
                   ↓
          Kalman Filter fusion
                   ↓
                  MCU
           ┌───────┴────────┐
           ↓                ↓
    Damper Actuators   Steering Servos
    (4 corners)        (Ackermann correction)
           ↓                ↓
    Optimised ride     Precise cornering
```

---

## Why This Matters Beyond Motorsports

The technologies developed in this project have broad applications:

| Technology | Broader Application |
|---|---|
| Predictive-reactive control | Autonomous vehicle suspension systems |
| Kalman filter sensor fusion | Robotics, UAVs, satellite attitude control |
| LQR optimal control | Industrial automation, aerospace systems |
| Real-time embedded MCU control | Medical devices, smart manufacturing |

Motorsports engineering has historically driven innovation that eventually reaches consumer vehicles — active suspension, traction control, and carbon fibre all originated in racing before becoming mainstream.

---

## Current Development Stage

This project is currently in **Phase 1 — Prototype and Simulation:**

- All structural components are 3D printed in **PETG-CF** for rapid iteration
- Mechanical design and CAD are complete
- Physical components have been acquired
- MATLAB/Simulink simulation is underway
- Full hardware assembly and Phase 2 material upgrade are planned once simulation validates the design

---

> 📝 Last updated: **March 2026**
