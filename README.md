# Design of a Servo Drive Control System for a Mobile Warehouse Sorting Robot

This repository archives the course project for Electric Drive Systems. The design target is a differential-drive servo drive control system for a mobile warehouse sorting robot. The full report is available at [`docs/course-design-report.pdf`](docs/course-design-report.pdf).

## Course Information

| Item | Content |
| --- | --- |
| School | Automation |
| Major | Automation |
| Student | Niyunjie |
| Student ID | 1320231081 |
| Project Title | Design of a Servo Drive Control System for a Mobile Warehouse Sorting Robot |
| Completion Date | May 17, 2026 |

## Project Overview

This project builds a load model for the differential drive wheels of a 50 kg mobile warehouse sorting robot, and completes motor and reducer selection, main drive circuit design, control strategy design, key component selection, cost estimation, and Simulink simulation verification. The system uses a 48 V low-voltage DC bus, a 48 V/150 W permanent magnet DC motor (PMDC), and a 17:1 planetary reducer. PWM voltage regulation is implemented through a MOSFET H-bridge, and a speed-current double closed-loop PI controller is used.

The design focuses on typical warehouse robot operating conditions, including precise low-speed docking, high-speed cruising, forward/reverse switching, downhill braking, emergency stopping, and load disturbance. The goal is to verify the dynamic response, safe braking capability, and engineering feasibility of the drive system.

## Naming Convention

Simulation files and result figures in this repository use English names:

- `math`: mathematical simulation model, built from the PMDC motor transfer function, equivalent load torque, back-EMF feedback, current loop, and speed loop.
- `physical-approx`: physical approximate simulation model, built with Simulink/Simscape PWM, H-bridge, motor, and feedback measurement blocks to approximate the real drive system.
- `speed`: speed response figure.
- `current`: current response figure.
- `tuned`: simulation result after controller parameter tuning in the downhill braking case.

## System Design

### Mechanical and Transmission Design

| Parameter | Value |
| --- | --- |
| Robot mass | 50 kg |
| Drive wheel radius | 0.08 m |
| Maximum slope | 5° |
| Rolling resistance coefficient | 0.04 |
| Mechanical transmission efficiency | 0.90 |
| Motor type | 48 V/150 W PMDC |
| Reducer | 17:1 planetary reducer |
| Rated motor speed | 5457 r/min |
| Rated motor torque | 0.26 N·m |
| Recommended peak torque | Not less than 0.60 N·m |
| Rated wheel-side torque | About 3.98 N·m |

### Main Circuit and Control Strategy

The main drive circuit consists of a 48 V DC bus, MOSFET H-bridge, PMDC motor, current sampling circuit, bus capacitor, and dynamic braking branch. The H-bridge adjusts the average armature voltage through PWM duty cycle control, enabling speed regulation, forward/reverse operation, and braking.

The control system uses a double closed-loop PI structure with an outer speed loop (ASR) and an inner current loop (ACR). The speed loop is responsible for speed tracking and dynamic response, while the current loop controls electromagnetic torque, current limiting, and impact suppression. The system also includes ramp starting, current-limited acceleration, safe reversal with deceleration before direction switching, and protection against overcurrent, overvoltage, undervoltage, stall, loss of speed, and overtemperature.

![Speed-current double closed-loop control structure](figures/speed-current-double-loop-control.png)

![MOSFET H-bridge structure](figures/mosfet-h-bridge.png)

## Simulink Models

This project builds both a mathematical simulation model and a physical approximate simulation model to compare the behavior of the double closed-loop PI controller under different modeling levels.

### Mathematical Simulation Model

Model file: [`models/math-simulation-model.slx`](models/math-simulation-model.slx)

![Mathematical simulation model Simulink diagram](simulation-results/math-simulation-model.png)

The mathematical simulation model describes the PMDC motor electrical dynamics, mechanical dynamics, and back-EMF feedback. It is suitable for controller parameter design, dynamic performance verification, and steady-state error analysis.

### Physical Approximate Simulation Model

Model file: [`models/physical-approx-simulation-model.slx`](models/physical-approx-simulation-model.slx)

![Physical approximate simulation model Simulink diagram](simulation-results/physical-approx-simulation-model.png)

The physical approximate simulation model adds a PWM Generator, H-bridge, motor module, and feedback measurement blocks. It is closer to the actual power drive system and can be used to observe the coupling among PWM voltage regulation, the power circuit, and the mechanical load.

## Simulation Cases and Results

Chapter 6 of the report analyzes six typical operating cases. The main evaluation metrics are rise time, speed overshoot, current overshoot, steady-state error, and energy flow. Overall, all cases basically meet the design requirements except the initial downhill braking parameters, which cause overshoot. After ASR/ACR parameter tuning, the speed and current overshoot in the downhill braking case are improved.

### Case 1: No-Load Fast Start Response

In the no-load case, the system directly steps to the command speed of 33.6 rad/s to verify the basic startup response of the double closed-loop controller. The report shows that both the mathematical simulation model and the physical approximate simulation model meet the rise time requirement of less than 0.8 s. The speed overshoot is controlled within 10%, the current overshoot is less than 5%, and the steady-state error is approximately zero.

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 1 mathematical model speed response](simulation-results/case-01-no-load-start-math-speed.svg) | ![Case 1 physical approximate model speed response](simulation-results/case-01-no-load-start-physical-approx-speed.svg) |
| ![Case 1 mathematical model current response](simulation-results/case-01-no-load-start-math-current.svg) | ![Case 1 physical approximate model current response](simulation-results/case-01-no-load-start-physical-approx-current.svg) |

### Case 2: Rated-Load Starting Performance

In the rated-load starting case, the system steps to 33.6 rad/s while applying a rated load torque of 0.09 N·m. This case verifies the motor starting capability under load and the current-limiting effect of the current loop. The simulation results show that both models reach the target speed within a short time, and the speed overshoot, current overshoot, and steady-state error all meet the requirements in the report.

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 2 mathematical model speed response](simulation-results/case-02-rated-load-start-math-speed.svg) | ![Case 2 physical approximate model speed response](simulation-results/case-02-rated-load-start-physical-approx-speed.svg) |
| ![Case 2 mathematical model current response](simulation-results/case-02-rated-load-start-math-current.svg) | ![Case 2 physical approximate model current response](simulation-results/case-02-rated-load-start-physical-approx-current.svg) |

### Case 3: Low-Speed to High-Speed Dynamic Switching

This case simulates the robot switching from low-speed docking to high-speed aisle cruising, with speed increasing from 7.33 rad/s to 33.6 rad/s. It corresponds to a typical warehouse robot scenario where the robot leaves a precise positioning area and enters a cruising aisle. The simulation results show that the system completes the speed transition smoothly, and the rise time, speed overshoot, current overshoot, and steady-state error all basically meet the requirements.

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 3 mathematical model speed response](simulation-results/case-03-low-to-high-speed-math-speed.svg) | ![Case 3 physical approximate model speed response](simulation-results/case-03-low-to-high-speed-physical-approx-speed.svg) |
| ![Case 3 mathematical model current response](simulation-results/case-03-low-to-high-speed-math-current.svg) | ![Case 3 physical approximate model current response](simulation-results/case-03-low-to-high-speed-physical-approx-current.svg) |

### Case 4: Fast Forward-Reverse Switching

In the forward-reverse switching case, the system switches from 33.6 rad/s to -33.6 rad/s while applying a rated load torque of 0.09 N·m. This case verifies the dynamic performance of the differential drive wheels during in-place turning, direction reversal, and reverse acceleration. The report adopts a safe reversal strategy of decelerating first, crossing zero speed, and then accelerating in the reverse direction. The simulation results show that the system can complete the forward-reverse switching process, the steady-state error is approximately zero, and the current impact is limited by the current loop.

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 4 mathematical model speed response](simulation-results/case-04-forward-reverse-math-speed.svg) | ![Case 4 physical approximate model speed response](simulation-results/case-04-forward-reverse-physical-approx-speed.svg) |
| ![Case 4 mathematical model current response](simulation-results/case-04-forward-reverse-math-current.svg) | ![Case 4 physical approximate model current response](simulation-results/case-04-forward-reverse-physical-approx-current.svg) |

### Case 5: Downhill Braking Characteristics

The downhill braking case simulates emergency braking from the cruising speed of 33.6 rad/s to 0 under a 5° downhill heavy-load condition. The corresponding load torque in the report is -0.28 N·m. In this case, the drive wheels back-drive the motor under gravity and inertia, causing the motor to enter generating braking mode. Mechanical energy and gravitational potential energy are converted into heat in the braking resistor through the H-bridge and braking branch.

With the initial controller parameters, the speed overshoot and current overshoot in downhill braking do not fully meet the requirements. The report then tunes the ASR/ACR parameters, after which the speed and current overshoot meet the design requirements.

**Initial parameter simulation results**

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 5 initial-parameter mathematical model speed response](simulation-results/case-05-downhill-braking-math-speed.svg) | ![Case 5 initial-parameter physical approximate model speed response](simulation-results/case-05-downhill-braking-physical-approx-speed.svg) |
| ![Case 5 initial-parameter mathematical model current response](simulation-results/case-05-downhill-braking-math-current.svg) | ![Case 5 initial-parameter physical approximate model current response](simulation-results/case-05-downhill-braking-physical-approx-current.svg) |

**Tuned parameter simulation results**

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 5 tuned-parameter mathematical model speed response](simulation-results/case-05-downhill-braking-math-speed-tuned.svg) | ![Case 5 tuned-parameter physical approximate model speed response](simulation-results/case-05-downhill-braking-physical-approx-speed-tuned.svg) |
| ![Case 5 tuned-parameter mathematical model current response](simulation-results/case-05-downhill-braking-math-current-tuned.svg) | ![Case 5 tuned-parameter physical approximate model current response](simulation-results/case-05-downhill-braking-physical-approx-current-tuned.svg) |

### Case 6: Disturbance Rejection Under a 20% Load Step

This case simulates a sudden 20% load torque increase while the robot cruises at 33.6 rad/s, from 0.09 N·m to 0.108 N·m. It verifies the disturbance rejection capability of the system under external load changes. The simulation results show that after the load step, the current loop quickly compensates for the load torque change, the speed loop pulls the speed back to the command value, and the steady-state error remains close to zero.

| Mathematical Simulation Model | Physical Approximate Simulation Model |
| --- | --- |
| ![Case 6 mathematical model speed response](simulation-results/case-06-load-disturbance-math-speed.svg) | ![Case 6 physical approximate model speed response](simulation-results/case-06-load-disturbance-physical-approx-speed.svg) |
| ![Case 6 mathematical model current response](simulation-results/case-06-load-disturbance-math-current.svg) | ![Case 6 physical approximate model current response](simulation-results/case-06-load-disturbance-physical-approx-current.svg) |

## Key Components and Cost

| Component | Recommended Specification | Quantity | Function |
| --- | --- | --- | --- |
| Drive motor | 48 V 150 W PMDC | 2 | Provides power for the left and right drive wheels |
| Reducer | 17:1 planetary reducer | 2 | Reduces speed and increases torque to match the maximum wheel speed |
| H-bridge driver module | BTS7960 or equivalent 48 V MOSFET H-bridge | 2 | PWM speed control, forward/reverse operation, and braking |
| Current sensor | ACS712-20A | 2 | Current-loop feedback and overcurrent protection |
| Speed feedback sensor | MT6701 magnetic encoder | 2 | Speed closed-loop feedback |
| Braking resistor | 5 Ω / 50 W aluminum-housed resistor | 4 | Dynamic braking during downhill operation and emergency stopping |
| Bus capacitor | 4700 μF / 63 V | 2 | Stabilizes the 48 V DC bus |

The report estimates the total hardware BOM cost at about 2770 CNY. The main costs are concentrated in the motors, reducers, and speed feedback sensors. The overall design uses mature industrial modules, making it suitable for prototyping, debugging, and later maintenance.

## File Structure

```text
.
├── .gitignore
├── README.md
├── README_cn.md
├── docs/
│   ├── course-design-report.pdf
│   └── course-design-report-source.docx
├── models/
│   ├── math-simulation-model.slx
│   └── physical-approx-simulation-model.slx
├── figures/
│   ├── drive-main-circuit-source.agx
│   ├── drive-main-circuit.svg
│   ├── speed-current-double-loop-control.png
│   ├── mosfet-h-bridge.png
│   ├── pmdc-motor.jpg
│   └── planetary-gearbox.jpg
└── simulation-results/
    ├── simulation-cases.txt
    ├── math-simulation-model.png
    ├── physical-approx-simulation-model.png
    ├── case-01-no-load-start-math-speed.svg
    ├── case-01-no-load-start-math-current.svg
    ├── case-01-no-load-start-physical-approx-speed.svg
    ├── case-01-no-load-start-physical-approx-current.svg
    ├── case-02-rated-load-start-math-speed.svg
    ├── case-02-rated-load-start-math-current.svg
    ├── case-02-rated-load-start-physical-approx-speed.svg
    ├── case-02-rated-load-start-physical-approx-current.svg
    ├── case-03-low-to-high-speed-math-speed.svg
    ├── case-03-low-to-high-speed-math-current.svg
    ├── case-03-low-to-high-speed-physical-approx-speed.svg
    ├── case-03-low-to-high-speed-physical-approx-current.svg
    ├── case-04-forward-reverse-math-speed.svg
    ├── case-04-forward-reverse-math-current.svg
    ├── case-04-forward-reverse-physical-approx-speed.svg
    ├── case-04-forward-reverse-physical-approx-current.svg
    ├── case-05-downhill-braking-math-speed.svg
    ├── case-05-downhill-braking-math-current.svg
    ├── case-05-downhill-braking-math-speed-tuned.svg
    ├── case-05-downhill-braking-math-current-tuned.svg
    ├── case-05-downhill-braking-physical-approx-speed.svg
    ├── case-05-downhill-braking-physical-approx-current.svg
    ├── case-05-downhill-braking-physical-approx-speed-tuned.svg
    ├── case-05-downhill-braking-physical-approx-current-tuned.svg
    ├── case-06-load-disturbance-math-speed.svg
    ├── case-06-load-disturbance-math-current.svg
    ├── case-06-load-disturbance-physical-approx-speed.svg
    └── case-06-load-disturbance-physical-approx-current.svg
```

## Usage

- Read the full design report: open [`docs/course-design-report.pdf`](docs/course-design-report.pdf).
- Edit the report source document: open [`docs/course-design-report-source.docx`](docs/course-design-report-source.docx).
- Reproduce the mathematical simulation: open [`models/math-simulation-model.slx`](models/math-simulation-model.slx) with MATLAB/Simulink.
- Reproduce the physical approximate simulation: open [`models/physical-approx-simulation-model.slx`](models/physical-approx-simulation-model.slx) with MATLAB/Simulink.
- View the simulation results: enter `simulation-results/` and use [`simulation-results/simulation-cases.txt`](simulation-results/simulation-cases.txt) to match each case number with its speed and current response figures.

## Notes

This repository is used for course project archiving, presentation, and simulation result reproduction. The model parameters, controller parameters, component selection, and cost estimation are all based on the course design analysis in the report.
