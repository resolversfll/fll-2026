# LEGO Education SPIKE Prime — Calibrated MyBlocks Reference

This repository documents the core Word Blocks **MyBlocks** used for **Robotina** in FIRST LEGO League (FLL) Challenge runs. Each custom block standardizes hardware initialization, proportional straight driving, calibrated gyro turns, and mat calibration routines.

---

## Hardware Configuration Summary

| Component | Port(s) | Configuration / Calibrated Value | Notes |
| :--- | :--- | :--- | :--- |
| **Left Drive Motor** | `A` | Paired as `A+E` | Large SPIKE wheel (88 mm diameter) |
| **Right Drive Motor** | `E` | Paired as `A+E` | Large SPIKE wheel (88 mm diameter) |
| **Wheel Circumference** | `A+E` | `27.6 cm` | Stored in `wheel_circumference` variable |
| **Attachment Motors** | `B`, `C` | Stop action: `hold position` | Prevents arms/claws from sagging |
| **Default `Kp` Gain** | Hub Gyro | `1.5` (tunable via input) | Proportional steering gain for `GyroStraight` |
| **Turn Calibration Factor** | Hub Gyro | `0.97` (`0.9694` measured) | Compensates for rotational momentum |

---

## 1. `Setup`

Initializes the drive base, wheel circumference, stop behavior, gyro sensor, motor encoders, and attachment hold modes before a mission begins.

### Inputs
* *None*

### Internal Block Logic
1. `set [wheel_circumference v] to [27.6]` *(calibrated from `VerifyDistance`)*
2. `set movement motors to [A+E v]`
3. `set 1 motor rotation to [wheel_circumference] [cm v] moved`
4. `set movement motors to [hold position v] at stop`
5. `set yaw angle to [0]`
6. `[A v] set relative position to [0]`
7. `[E v] set relative position to [0]`
8. `[B v] set motor to [hold position v] at stop`
9. `[C v] set motor to [hold position v] at stop`

### Usage Example
Place **`Setup`** directly under `when program starts` at the beginning of every mission stack.

```text
when program starts
└── Setup
```

---

## 2. `GyroStraight (distance_cm) (speed) (Kp)`

Drives the robot straight for an exact distance in centimeters while continuously correcting heading drift using the hub's yaw angle sensor.

### Inputs
* **`distance_cm`** *(Number)*: Target distance to drive in centimeters.
* **`speed`** *(Number)*: Base motor speed percentage (`-100` to `100`; use `30–40%` for precision approaches, `50–60%` for long transits).
* **`Kp`** *(Number)*: Proportional steering gain (default **`1.5`**). Controls how strongly the robot corrects off-course drift.

### Internal Block Logic
1. `set yaw angle to [0]`
2. `[A v] set relative position to [0]`
3. `set [target_rotations v] to [(distance_cm) / (wheel_circumference)]`
4. `repeat until <([abs v] of ([A v] relative position)) > ((target_rotations) * (360))>`
   * `set [correction v] to (((0) - ([yaw angle])) * (Kp))`
   * `start moving at ((speed) + (correction)) ((speed) - (correction)) % speed`
5. `stop moving`

### Usage Examples
* **Precision approach (`45 cm` at `35%` speed with `Kp = 1.5`):**
  ```text
  GyroStraight (45) (35) (1.5)
  ```
* **Fast transit across the mat (`110 cm` at `60%` speed with `Kp = 1.3`):**
  ```text
  GyroStraight (110) (60) (1.3)
  ```

---

## 3. `GyroTurn (degrees) (speed) (calibration_factor)`

Executes an accurate point turn to the right (positive degrees) or left (negative degrees) using a calibration multiplier to stop cleanly without rotational overshoot.

### Inputs
* **`degrees`** *(Number)*: Target turn angle (`+` for right turns, `-` for left turns). Keep single turns `<= 180°` to avoid yaw wrap-around at `±179°`.
* **`speed`** *(Number)*: Turn speed percentage (recommended **`25–35%`** for consistent braking).
* **`calibration_factor`** *(Number)*: Scale factor determined by `FindCalibrationFactor` (default **`0.97`** for Robotina).

### Internal Block Logic
1. `set yaw angle to [0]`
2. `set movement speed to [speed] %`
3. `set [target v] to ((degrees) * (calibration_factor))`
4. `if <(target) > [0]> then` *(Right Turn)*
   * `start moving [right: 100]`
   * `wait until <([yaw angle]) > (target)>`
5. `else` *(Left Turn)*
   * `start moving [left: -100]`
   * `wait until <([yaw angle]) < (target)>`
6. `stop moving`

### Usage Examples
* **Turn 90° Right:**
  ```text
  GyroTurn (90) (30) (0.97)
  ```
* **Turn 90° Left:**
  ```text
  GyroTurn (-90) (30) (0.97)
  ```
* **Turn 45° Right:**
  ```text
  GyroTurn (45) (25) (0.97)
  ```

---

## 4. `VerifyDistance (speed) (rotations)`

Calibration helper block used to measure real-world wheel travel over a fixed number of motor rotations so you can calculate exact `wheel_circumference`.

### Inputs
* **`speed`** *(Number)*: Test speed percentage (recommended **`30%`** to prevent wheel slip).
* **`rotations`** *(Number)*: Number of wheel rotations to drive (recommended **`5`**).

### Internal Block Logic
1. `set movement motors to [A+E v]`
2. `set movement speed to [speed] %`
3. `move [straight: 0] for (rotations) [rotations v]`

### Usage Example
```text
when program starts
└── VerifyDistance (30) (5)
```

---

## 5. `FindCalibrationFactor (speed) (yaw_threshold)`

Calibration helper block used to spin the robot until the hub reaches `yaw_threshold`, allowing you to tune out rotational momentum overshoot.

### Inputs
* **`speed`** *(Number)*: Turn speed percentage (recommended **`10–30%`**).
* **`yaw_threshold`** *(Number)*: Raw gyro stop angle (`<= 178°` to stay inside the hub's `-179°` to `+179°` yaw range; e.g., `87` for a `90°` test or `175` for a `180°` test).

### Internal Block Logic
1. `set movement motors to [A+E v]`
2. `set movement speed to [speed] %`
3. `set yaw angle to [0]`
4. `start moving [right: 100]`
5. `wait until <([yaw angle]) > (yaw_threshold)>`
6. `stop moving`

### Usage Example
```text
when program starts
└── FindCalibrationFactor (10) (175)
```
*(Note: Keep `yaw_threshold` at `178` or lower—the SPIKE Prime yaw sensor wraps around from `+179°` to `-180°` and will never trigger a threshold above `179`.)*

---

## Complete Sample Mission Stack

```text
when program starts
├── Setup
├── GyroStraight (60) (45) (1.5)
├── GyroTurn (90) (30) (0.97)
├── GyroStraight (35) (35) (1.5)
├── [B v] run [clockwise v] for (140) [degrees v]
├── GyroTurn (-45) (30) (0.97)
└── GyroStraight (-50) (50) (1.5)
```
