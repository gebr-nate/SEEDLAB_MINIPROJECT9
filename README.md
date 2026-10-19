# Mini Project: Vision-Guided Wheel Positioning

## Objective
Build a system in which a camera-detected ArUco marker commands the angular
position of two motorized wheels. Moving the marker between the four quadrants
of the camera image must make each wheel rotate to one of two positions
(0 or half a revolution), and the LCD must display the commanded goal.

### Specific goals
1. **Detect** an ArUco marker (DICT_6X6_50) in real time at 320x240, 30 FPS.
2. **Classify** the marker center into one of four image quadrants.
3. **Communicate** the resulting two-bit wheel goal to the Arduino over USB serial.
4. **Control** each wheel to its goal position using closed-loop feedback from
   its encoder, with a velocity limit and anti-windup.
5. **Display** the current goal on a 16x2 character LCD without slowing the
   vision loop.
6. **Model** the motor and controller in Simulink to choose and verify gains.
7. **Verify** each subsystem independently before integration.

[Edit these to match the assignment's official requirements and any numeric
specs, such as settling time or overshoot.]

## Quadrant to Wheel Goal Mapping
Image is 320x240; dividing lines at x = 160 and y = 120.

| Quadrant | Marker center | left_goal | right_goal | Command sent |
|----------|---------------|-----------|------------|--------------|
| NW | x < 160, y < 120   | 0 | 1 | `10` |
| NE | x >= 160, y < 120  | 0 | 0 | `00` |
| SW | x < 160, y >= 120  | 1 | 1 | `11` |
| SE | x >= 160, y >= 120 | 1 | 0 | `01` |

A goal bit of 0 means 0 rad; 1 means PI rad (half a wheel revolution).

## Serial Protocol (Pi to Arduino)
- 115200 baud, port `/dev/ttyACM0` on the Pi.
- One message per goal change: two ASCII characters `0`/`1` followed by `\n`.
- Format `<right_goal><left_goal>`. The first character drives motor 1
  (right wheel) and the second drives motor 2 (left wheel).
- Invalid messages are ignored. A valid message resets the position-loop
  integrators.

## Code Organization
| Folder | Role |
|--------|------|
| [`Arduino/`](Arduino/) | Real-time motor control (100 Hz cascaded controller) |
| [`Raspberry Pi/`](Raspberry%20Pi/) | Vision, quadrant logic, LCD, serial sender |
| [`Simulink/`](Simulink/) | Models used to design and tune the controller |
| [`Subsystem Tests/`](Subsystem%20Tests/) | Stand-alone tests for each subsystem |

Execution flow at run time:
1. The Pi grabs a frame, converts it to grayscale, and detects markers.
2. The first detected marker's center selects a quadrant and goal pair.
3. If the goal changed, the Pi queues it for the LCD thread and sends the
   command to the Arduino.
4. The Arduino updates its goals and, every 10 ms, runs the position and
   velocity loops for both motors.
5. If the marker is lost, the last goal is held.

## Control Design Summary
Cascaded loops on each wheel, running every 10 ms:

| Loop | Type | Gain | Output |
|------|------|------|--------|
| Outer (position) | PI | Kp_pos = 5.0 (rad/s)/rad, Ki_pos = 0.5 | desired speed, clamped to +/- 8 rad/s, with anti-windup |
| Inner (velocity) | P | Kp = 3 V/(rad/s) | motor voltage, converted to PWM using a 7.6 V supply |

Encoder resolution is 800 counts per wheel revolution (the ISR counts the rising
edge of channel A only; direction comes from channel B).

## Hardware and Wiring
| Connection | Details |
|------------|---------|
| Pi to Arduino | USB cable (serial) |
| Pi to LCD | I2C (SDA, SCL, power, GND) |
| Pi to camera | USB, device index 0 |
| Arduino to motor driver | Enable D4; direction D7/D8; PWM D9/D10 |
| Arduino to encoders | Channel A on D2/D3 (interrupts); channel B on D5/D6 |
| Power | About 7.6 V battery to motor driver; common ground everywhere |

Motor 2 is physically mounted in reverse; its direction logic and encoder sign
are inverted in software so positive motion is forward for both wheels.

## How to Run
1. Upload the sketch in `Arduino/` (see its README).
2. Connect the Arduino to the Pi by USB and power the motors.
3. On the Pi: `python3 "Raspberry Pi/aruco_quadrant_wheel_control.py"`
4. Hold the marker in each quadrant and confirm the wheels and LCD respond.
5. Press `q` in the preview window to stop.

To test the Arduino alone, open the Serial Monitor at 115200 baud (Newline
line ending) and send `00`, `01`, `10`, or `11`.

## Known Limitations
- Only the first detected marker is used.
- Wheel zero is wherever the wheels sit at Arduino power-up or reset.
- The Arduino does not report position back to the Pi.
- Commands are sent only when the goal changes, so a dropped serial write is
  not retried until the marker moves to a different quadrant.
