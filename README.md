# 🏎️ F1-Inspired Semi-Active Suspension System with LIDAR Predictive Control

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Domain](https://img.shields.io/badge/Domain-Mechatronics-blue)
![Control](https://img.shields.io/badge/Control-PID%20%7C%20LQR%20%7C%20Kalman%20Filter-green)
![Suspension](https://img.shields.io/badge/Suspension-Semi--Active%20Double%20Wishbone-orange)

## 📌 Overview

This project presents the design and development of an **F1-inspired semi-active suspension system** integrating **LIDAR-based predictive terrain response** and **dual-servo independent steering** for motorsports vehicles.

Unlike conventional reactive suspension systems, this system uses a **parallel predictive-reactive control loop** — combining real-time sensor feedback with forward-looking terrain prediction — to achieve smoother ride, consistent downforce, and faster recovery from surface changes.

The system architecture mirrors modern Formula 1 engineering principles, scaled and adapted for research and prototyping purposes.

---

## 🎯 Key Features

- **F1-style double wishbone pushrod suspension** with electronically controlled semi-active damping
- **LIDAR predictive mechanism** — scans terrain 1.5–2m ahead and pre-adjusts suspension before surface contact
- **Parallel predictive-reactive control loop** using data-fusion (weighted PID / Kalman filter)
- **Dual-servo independent steering** replicating F1 Ackermann geometry for tighter cornering
- **MCU-integrated sensor array** — IMU, ride-height sensors, displacement/pressure sensors
- **Response time < 10ms** for real-time damping modulation

---

## 🔧 System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    SENSOR ARRAY                         │
│  LIDAR (Predictive) │ IMU │ Ride-Height │ Displacement  │
└────────────┬────────────────────────┬────────────────────┘
             │                        │
     Predictive Path           Reactive Path
             │                        │
             ▼                        ▼
    ┌─────────────────────────────────────┐
    │       DATA FUSION ALGORITHM         │
    │   Weighted PID / Kalman Filter      │
    │  (Higher confidence signal wins)    │
    └──────────────────┬──────────────────┘
                       │
                       ▼
            ┌──────────────────┐
            │       MCU        │
            └────────┬─────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
  Damper Actuator          Steering Servos
  (Stiffness Control)      (Ackermann Correction)
```

---

## 🏗️ Suspension Specifications

| Parameter | Specification | Description |
|---|---|---|
| **Type** | Independent Double Wishbone, Pushrod & Rocker | F1-inspired design |
| **Operation Mode** | Semi-Active (Electronically Controlled Damping) | MCU-regulated |
| **Material (Prototype)** | PETG-CF (Carbon Fibre reinforced PETG) | Rapid iteration, stiff, lightweight |
| **Material (Final — Planned)** | CF-Nylon arms, CNC Aluminium rockers, Steel coilovers | Motorsports-grade strength and durability |
| **Spring Rate (Front)** | 2.2 – 2.5 N/mm | Precision turn-in |
| **Spring Rate (Rear)** | 1.8 – 2.2 N/mm | Traction stability |
| **Damper Stroke** | 25 – 30 mm | Tuneable via rocker ratio |
| **Shock Oil** | 45 – 60 wt. Silicone | Variable damping control |
| **Actuator Power** | 4 – 6 W Micro Linear Actuator | Electronic stiffness control |
| **Response Time** | < 10 ms | Instant damping modulation |
| **Ride Height Range** | 6 – 14 mm (active adjustment) | Electronically controlled |
| **Total Chassis Weight** | 7.0 – 7.5 kg | Including all electronics |

---

## 📡 LIDAR Predictive-Reactive Control

The core innovation of this system is a **dual-path parallel control loop**:

### Control Flow:
1. **LIDAR Scanning** — Solid-state LIDAR maps surface irregularities (bumps, dips, curbs) within 1.5–2m range
2. **Reactive Feedback** — IMU and wheel sensors gather real-time vertical acceleration and compression data simultaneously
3. **Data Fusion** — Weighted PID / Kalman filter evaluates which input has higher confidence at each moment
4. **Adaptive Execution:**
   - High LIDAR confidence → suspension pre-loads **before** surface contact (predictive)
   - Low LIDAR confidence → reactive actuator feedback takes priority (safe fallback)

### Performance Outcomes:
- Minimised impact shock and pitch
- Consistent downforce under varying terrain
- Faster recovery from surface changes
- Improved high-speed stability

---

## 🔩 Mechanical Layout

```
Front/Rear Assembly:
Wheel → Upper & Lower Wishbones → Pushrod → Rocker Arm
→ Horizontal Coilover (Spring + Damper + Micro Linear Actuator)
→ MCU ← Sensor Array (IMU, LIDAR, Ride-Height, Displacement)
→ Actuator Command → Real-time Damper Stiffness Control
```

### Actuator Configuration (Under R&D)

| Option | Description | Assessment |
|---|---|---|
| **Option 1 (Preferred)** | Physical damper + actuator damper mounted in parallel to rocker | Preferred — balanced performance |
| **Option 2** | Single actuator damper replacing physical damper entirely | Possible but power-hungry and heat-intensive |

> Final configuration to be determined through simulation and prototype testing.

---

## 🎮 Dual-Servo Independent Steering

| Parameter | Specification | Description |
|---|---|---|
| **Type** | Dual Independent Servo Steering | Simulates F1 Ackermann geometry |
| **Servo Torque** | 30 – 35 kg·cm (metal gear, high-speed) | Responsive and precise |
| **Control Logic** | MCU-synchronized dual input | Differential steering angle calculation |
| **Steering Ratio** | ~15:1 (adjustable) | Tight F1-style cornering |
| **Feedback** | Potentiometer / Magnetic encoder | Real-time wheel-angle monitoring |

Two independent servos actuate left and right steering arms separately. The MCU applies Ackermann correction in real time, reducing understeer and replicating F1 front-end dynamics.

---

## 🧠 Control Strategies

### PID Controller
- Classical closed-loop feedback control
- Tuned for minimising body displacement and vertical acceleration
- Baseline controller for performance benchmarking

### LQR (Linear Quadratic Regulator)
- State-space optimal control approach
- Minimises cost function balancing ride comfort vs road handling
- Expected to outperform PID under dynamic motorsports conditions

### Kalman Filter (Sensor Fusion)
- Fuses LIDAR predictive data with reactive actuator feedback
- Continuously determines higher-confidence signal
- Enables smooth transition between predictive and reactive modes

---

## 📊 Current Project Status

| Phase | Status |
|---|---|
| System concept & design | ✅ Completed |
| Suspension architecture definition | ✅ Completed |
| Mechanical design & specifications | ✅ Completed |
| CAD modelling | ✅ Completed |
| Physical components acquisition | ✅ Completed |
| Actuator configuration finalisation | 🔄 R&D In Progress |
| MATLAB/Simulink simulation | 🔄 In Progress |
| PID controller implementation | 🔄 In Progress |
| LQR controller implementation | ⏳ Planned |
| Kalman filter & sensor fusion | ⏳ Planned |
| Hardware assembly & integration | ⏳ Planned |
| LIDAR predictive loop integration | ⏳ Planned |
| Real-world testing & validation | ⏳ Planned |

---

## 🛠️ Tools & Technologies

| Category | Tools |
|---|---|
| **CAD Design** | Fusion 360 |
| **Simulation** | MATLAB |
| **Control Algorithms** | PID, LQR, Kalman Filter |
| **Microcontroller** | Teensy 4.0 |
| **Sensors** | IMU, Solid-state LIDAR, Ride-height sensors, Displacement sensors |
| **Actuators** | Micro Linear Actuators (4–6W), Dual Metal-Gear Servos (30–35 kg·cm) |
| **Prototype Materials** | PETG-CF (Carbon Fibre reinforced PETG) — all structural components |
| **Final Materials (Planned)** | CF-Nylon, CNC Aluminium, Steel coilovers |

---

## 📁 Repository Structure

```
active-suspension-f1/
├── README.md
├── docs/
│   ├── system-overview.md
│   ├── design-decisions.md
│   ├── control-strategy.md
│   └── progress-log.md
├── cad/
│   └── [CAD model files]
├── simulations/
│   ├── quarter_car_model/
│   └── control_comparison/
├── code/
│   ├── pid_controller/
│   ├── lqr_controller/
│   ├── kalman_filter/
│   └── sensor_fusion/
├── data/
│   └── [Simulation and test data]
└── media/
    └── [Photos, diagrams, videos]
```

---

## 📈 Progress Log

### March 2026
- Completed full system design and mechanical specifications
- Finalised F1-inspired double wishbone pushrod architecture
- Completed CAD modelling
- Acquired physical components
- Currently evaluating actuator configuration — parallel vs replacement approach
- MATLAB quarter-car simulation in progress

> Updated monthly as the project develops.

---

## 📚 References

- Hrovat, D. (1997). Survey of Advanced Suspension Developments and Related Optimal Control Applications. *Automatica*
- Rajamani, R. (2012). *Vehicle Dynamics and Control*. Springer
- Williams, R.A. (1994). Automotive Active Suspensions. *Proceedings of the Institution of Mechanical Engineers*
- MATLAB Control System Toolbox Documentation

---

## 👤 Author

**[Pawar Adwait Satish]**
B.Eng. Mechatronics & Robotics — Kyoto University of Advanced Science (KUAS)
[LinkedIn](www.linkedin.com/in/adwaitpawar-astrox) | [Email](astroxindia@gmail.com)

---

> ⚙️ This project is part of ongoing undergraduate research in mechatronics, control systems, and autonomous vehicle dynamics.
