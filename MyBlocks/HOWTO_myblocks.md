# LEGO Education SPIKE Prime MyBlocks Documentation

A collection of reusable **SPIKE Prime Word Blocks (MyBlocks)** designed for **FIRST LEGO League (FLL)** competition consistency, drive-base calibration, proportional gyro straight driving, and calibrated gyro turns.

---

## Table of Contents

- [Hardware & Configuration Overview](#hardware--configuration-overview)
- [Core Mission MyBlocks](#core-mission-myblocks)
  - [1. `Setup`](#1-setup)
  - [2. `GyroStraight`](#2-gyrostraight)
  - [3. `GyroTurn`](#3-gyroturn)
- [Calibration & Testing MyBlocks](#calibration--testing-myblocks)
  - [4. `VerifyDistance`](#4-verifydistance)
  - [5. `FindCalibrationFactor`](#5-findcalibrationfactor)
- [Step-by-Step Calibration Workflow](#step-by-step-calibration-workflow)
- [Complete Mission Example](#complete-mission-example)
- [Troubleshooting & Competition Tips](#troubleshooting--competition-tips)

---

## Hardware & Configuration Overview

| Parameter | Default / Configured Value | Notes |
| :--- | :--- | :--- |
| **Drive Motors** | `A+E` | Left and right drive wheels paired in `A+E` order |
| **Wheel Size** | Large SPIKE Prime Wheel (`88 mm` diameter) | Theoretical circumference: `27.6 cm` |
| **Stop Action** | `hold position` | Actively locks motors at the end of moves/turns to prevent coasting |
| **Steering Gain (`Kp`)** | `1.5` | Proportional correction multiplier used in `GyroStraight` |
| **Turn Calibration Factor** | `0.97` | Momentum compensation scale factor used in `GyroTurn` |

---

## Core Mission MyBlocks

### 1. `Setup`

Initializes the drive base, sets the calibrated wheel circumference, locks motor stop behavior, and zeroes out the hub's yaw sensor and motor relative encoders before a mission starts.

#### Parameters
*None.*

#### Block Logic
```text
define Setup
├── set [wheel_circumference] to (27.6)
├── set movement motors to [A+E]
├── set 1 motor rotation to (wheel_circumference) cm moved
├── set movement motors to [hold position] at stop
├── set yaw angle to (0)
├── [A] set relative position to (0)
└── [E] set relative position to (0)
```

#### How to Use
Place `Setup` as the **very first block** directly beneath `when program starts` in every mission program.

```text
when program starts
└── Setup
```

---

### 2. `GyroStraight`

Drives the robot straight for an exact distance in centimeters while continuously correcting heading drift using proportional feedback from the hub's internal yaw sensor.

#### Parameters
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `distance_cm` | Number | Target travel distance in centimeters (`cm`). |
| `speed` | Number | Base motor power percentage (`0` to `100`). Recommended: `30`–`50` for high accuracy. |

#### Block Logic
```text
define GyroStraight (distance_cm) (speed)
├── set yaw angle to (0)
├── [A] set relative position to (0)
├── set [target_rotations] to ((distance_cm) / (wheel_circumference))
├── repeat until <(abs of ([A] relative position)) > ((target_rotations) * (360))>
│   ├── set [correction] to (((0) - (yaw angle)) * (1.5))
│   └── start moving at ((speed) + (correction)) ((speed) - (correction)) % speed
└── stop moving
```

#### How to Use
Call `GyroStraight` whenever you need reliable, straight-line travel across the mat without veering off course.

```text
# Drive straight forward 45 cm at 40% speed
GyroStraight (45) (40)
```

---

### 3. `GyroTurn`

Executes an in-place pivot turn to a target angle (both clockwise and counter-clockwise) using the hub's yaw sensor and a momentum calibration factor so the robot stops squarely on target without overshooting.

#### Parameters
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `degrees` | Number | Target turn angle in degrees. Use **positive** values (e.g., `90`) to turn right and **negative** values (e.g., `-90`) to turn left. Keep single turns under `180°`. |
| `speed` | Number | Turn motor speed percentage (`0` to `100`). Recommended: `25`–`35` for crisp stops. |

#### Block Logic
```text
define GyroTurn (degrees) (speed)
├── set yaw angle to (0)
├── set movement speed to (speed) %
├── set [target] to ((degrees) * (0.97))
├── if <(target) > (0)> then
│   ├── start moving [right: 100]
│   └── wait until <(yaw angle) > (target)>
├── else
│   ├── start moving [left: -100]
│   └── wait until <(yaw angle) < (target)>
└── stop moving
```

#### How to Use
Pass positive angles for right turns and negative angles for left turns.

```text
# Turn 90 degrees right at 30% speed
GyroTurn (90) (30)

# Turn 45 degrees left at 30% speed
GyroTurn (-45) (30)
```

---

## Calibration & Testing MyBlocks

Use these two utility MyBlocks on your competition mat whenever you change robot weight, swap wheels, or notice slight drift.

### 4. `VerifyDistance`

Drives the robot straight for an exact number of motor rotations so you can measure physical travel and calculate your robot's true `wheel_circumference`.

#### Parameters
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `speed` | Number | Movement speed percentage (use `30` to prevent wheel slip). |
| `rotations` | Number | Number of full wheel rotations to travel (e.g., `5`). |

#### Block Logic
```text
define VerifyDistance (speed) (rotations)
├── set movement motors to [A+E]
├── set movement speed to (speed) %
└── move [straight: 0] for (rotations) rotations
```

#### How to Use
```text
when program starts
└── VerifyDistance (30) (5)
```

---

### 5. `FindCalibrationFactor`

Performs a raw, unscaled right pivot turn up to `yaw_threshold` so you can measure momentum overshoot and compute the exact calibration multiplier for `GyroTurn`.

#### Parameters
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `speed` | Number | Turn speed percentage (match the speed you plan to use in missions, e.g., `30`). |
| `yaw_threshold` | Number | Raw yaw angle (`< 179`) at which the motors cut power. Start with `87` for a `90°` turn test. |

#### Block Logic
```text
define FindCalibrationFactor (speed) (yaw_threshold)
├── set movement motors to [A+E]
├── set movement speed to (speed) %
├── set yaw angle to (0)
├── start moving [right: 100]
├── wait until <(yaw angle) > (yaw_threshold)>
└── stop moving
```

#### How to Use
```text
when program starts
└── FindCalibrationFactor (30) (87)
```

---

## Step-by-Step Calibration Workflow

### Step 1: Calibrate `wheel_circumference`
1. Place your robot on the FLL mat with a fixed reference point (such as the front axle or a LEGO pointer) aligned on a tape mark.
2. Run `VerifyDistance (30) (5)`.
3. Measure the exact distance traveled in centimeters (e.g., `137.5 cm`).
4. Divide the measured distance by `5`:
   ```text
   wheel_circumference = Measured Distance (cm) / 5
   # Example: 137.5 / 5 = 27.5 cm
   ```
5. Update `set [wheel_circumference] to (...)` inside your **`Setup`** MyBlock with this value.

### Step 2: Calibrate Turn Multiplier (`GyroTurn`)
1. Align a straight edge of your robot squarely against a line or border on the mat.
2. Run `FindCalibrationFactor (30) (87)`.
3. Observe where the robot comes to rest relative to a true `90°` perpendicular line:
   - **Overshoots `90°`:** Lower `yaw_threshold` (e.g., try `85` or `86`).
   - **Undershoots `90°`:** Raise `yaw_threshold` (e.g., try `88` or `89`).
4. Once the robot lands squarely at `90°`, divide your winning `yaw_threshold` by `90`:
   ```text
   Calibration Factor = Winning yaw_threshold / 90
   # Example: 87 / 90 = 0.967 (round to 0.97)
   ```
5. Update the multiplier (`0.97`) in `set [target] to ((degrees) * (0.97))` inside your **`GyroTurn`** MyBlock.

---

## Complete Mission Example

Below is an example of chaining `Setup`, `GyroStraight`, and `GyroTurn` together to drive out of base, turn toward a mission model, approach it, and return.

```text
when program starts
├── Setup
├── GyroStraight (60) (45)     # Drive 60 cm straight out of base at 45% speed
├── GyroTurn (90) (30)         # Turn 90 degrees right at 30% speed
├── GyroStraight (25) (35)     # Approach mission model 25 cm at 35% speed
├── GyroStraight (25) (-35)    # Back away 25 cm (or turn around)
├── GyroTurn (-90) (30)        # Turn 90 degrees left to face home
└── GyroStraight (60) (50)     # Return 60 cm to base
```

---

## Troubleshooting & Competition Tips

- **Yaw Sensor `180°` Wrap-Around:** The SPIKE Prime `yaw angle` sensor reports values from `-179°` to `+179°`. Never pass a `degrees` value of `180` or higher into `GyroTurn` or `FindCalibrationFactor`—for a `180°` turn, chain two `GyroTurn (90) (30)` blocks instead.
- **Steering Direction in `GyroStraight`:** If nudging the robot right causes it to curve *further* right instead of correcting back left, swap the `+` and `-` operators in `start moving at ((speed) + (correction)) ((speed) - (correction)) % speed`.
- **Tuning `Kp` (`1.5`) in `GyroStraight`:**
  - If the robot oscillates or wiggles side-to-side while driving straight, lower `1.5` to `1.0` or `1.2`.
  - If the robot drifts off line and corrects too slowly, raise `1.5` to `1.8` or `2.0`.
- **Consistent Motor Pairing:** Ensure every block uses `A+E` in the exact same order; reversing it to `E+A` inverts forward/backward and left/right directions.
