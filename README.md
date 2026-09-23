# SEEDLAB_MINIPROJECT9
Repository for **EENG350: Systems Exploration, Engineering, and Design Laboratory** at Colorado School of Mines (Spring 2026). It holds all code, models, and documentation for our two-wheeled mobile robot, which combines a Raspberry Pi (computer vision, user interface) with an Arduino (motor control).

## Team Members
| Name | Primary Role |
|------|--------------|
| [Nathan Gebre-Hiwot] | [Controls / Arduino] |
| [Bryce Lapham] | [Controls / Arduino] |
| [Jake Toschik] | [Computer Vision] |
| [Megan Brady] | [Computer Vision] |

## Repository Organization
| Folder | Contents |
|--------|----------|
| [`Mini Project/`](Mini%20Project/) | Computer-vision-driven wheel position control: Pi code, Arduino code, Simulink models, and subsystem test code. See its README for details. |

Each project folder has its own `README.md` describing its code, hardware connections, and how to run it.

## Hardware Overview
- Raspberry Pi with camera and Adafruit 16x2 character LCD
- Arduino with motor driver shield
- Two Pololu 37D gear motors (50:1) with quadrature encoders
