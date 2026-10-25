# Flying Wing Flight Stabilizer (STM32F303K8 + ELRS)

A small flight stabilizer for a foam flying wing, written in C for the **STM32F303K8** (Nucleo-32). The firmware reads an IMU, estimates roll and pitch, runs a control loop, drives two elevon servos and a brushless motor, and receives pilot commands from an **ExpressLRS (ELRS)** receiver over the **CRSF** protocol.

> University project for the *Embedded Systems* course. The goal is to learn embedded programming in C on STM32: timers, interrupts, serial communication, sensor handling, and real-time control.

---

## Table of contents

1. [Project goal](#1-project-goal)
2. [Scope](#2-scope)
3. [System overview](#3-system-overview)
4. [Hardware](#4-hardware)
5. [Team and responsibilities](#5-team-and-responsibilities)
6. [Learning outcomes](#6-learning-outcomes)

---

## 1. Project goal

Design and implement a stabilization system for a small flying wing. The STM32 uses accelerometer and gyroscope data to stabilize the aircraft's attitude, and the pilot controls it with an ELRS transmitter.

**Mandatory result:** stabilization of the aircraft on a test stand, and an evaluation of the response to a deflection (rise time, overshoot, settling time).

**Extension:** a short piloted flight with stabilization enabled.

The main purpose is **educational**. The firmware is written from scratch on top of the STM32 HAL/LL drivers, with no ready-made flight controller software (Betaflight, ArduPilot, and similar).

## 2. Scope

### Mandatory

- IMU driver (I²C) with gyro bias calibration
- Roll and pitch estimation (complementary filter)
- Fixed-rate control loop driven by a timer interrupt
- PID (or PD) control of roll and pitch
- Elevon mixer for two servos
- Motor control through an ESC (PWM), with safe arming
- CRSF reception from the ELRS receiver (UART)
- Flight modes: Manual, Stabilized, Failsafe
- Failsafe on loss of the radio link

## 3. System overview

```
                        +-----------------------------+
 ELRS transmitter  ~~~> |  ELRS receiver              |
                        +--------------+--------------+
                                       | CRSF, UART, 420 kbaud
                                       v
 +-----------+   I2C   +-------------------------------+   PWM    +--------------+
 |  MPU-6050 | <-----> |        STM32F303K8            | -------> | Servo (left) |
 |  IMU      |         |                               | -------> | Servo (right)|
 +-----------+         |  CRSF parser                  | -------> | ESC + motor  |
                       |  Attitude estimation          |          +--------------+
                       |  PID control + mixer          |
                       |  State machine, failsafe      |   UART
                       |  Logger                       | -------> PC (logging)
                       +-------------------------------+
```

**Data flow:** the timer interrupt triggers the control loop. In each cycle the firmware read
s the IMU, updates the attitude estimate, reads the latest pilot command, computes the control output, mixes it into elevon commands, and updates the PWM outputs.

## 4. Hardware

| Part | Example / specification | Purpose |
|---|---|---|
| Microcontroller board | Nucleo-32 with STM32F303K8 | Main controller |
| IMU | MPU-6050 (GY-521 module), I²C | Gyroscope and accelerometer |
| Radio receiver | ExpressLRS receiver with CRSF output | Pilot commands |
| Radio transmitter | ELRS-capable transmitter | Pilot input |
| Airframe | Foam flying wing, about 600-900 mm span | Test platform |
| Servos | 2 × 9 g micro servo | Elevons |
| Motor | Small brushless motor with pusher propeller | Propulsion |
| ESC | 12 A ESC with built-in 5 V BEC | Motor control and 5 V supply |
| Battery | 2S LiPo, 450-850 mAh | Power |
| Wiring | Wires, connectors, breadboard or perfboard | Integration |
| Test stand | Wooden or cardboard frame with a pivot axis | Stand tests |

Optional: BMP280 barometer, voltage divider for the battery (resistors).


## 5. Team and responsibilities

Each member owns one area and its interface, and presents it in the report and demo. Members review each other's code and explain their module to one teammate.

| Member | Area | Main tasks |
|---|---|---|
| Danyil Yatluk  | Sensing and estimation | IMU driver, calibration, filtering, attitude estimation |
| Stanislav Sharov| Control | PID, mixer, tuning, step-response measurements |
| Vasko Mykhailo | Actuation and hardware | Servo and ESC PWM, power, wiring, airframe,, ADC |
| Arman Vardanyan| Radio and system | CRSF parser, state machine, failsafe, logging, plotting, integration |

## 6. Learning outcomes

Through this project the team practices:

- GPIO, external and timer interrupts
- Timer PWM generation (prescaler, auto-reload, compare math)
- UART, I²C communication, and writing a protocol parser with CRC
- Reading datasheets and register maps
- Sensor calibration, digital filtering, and sensor fusion
- PID control, anti-windup, and experimental tuning
- Real-time design: fixed-rate loops, ISR and main-loop data sharing, failsafe design
- Debugging with SWD, UART logs, and a logic analyzer
- Teamwork with Git, module interfaces, and documentation
