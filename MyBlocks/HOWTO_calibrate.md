# SPIKE Prime Robot Calibration Guide — Movement & Attachment Motors

This guide provides step-by-step procedures for calibrating **movement motors (`A+E`)** and **attachment motors (`B` & `C`)** on **Robotina** so runs stay repeatable across battery levels and tournament tables.

---

## Part 1: Identifying & Verifying Motor Ports

Before calibrating, verify which ports control your drive wheels and attachments:

1. Connect the SPIKE Prime Hub to the app and open the **Hub Dashboard**.
2. **Drive Wheels:** Roll the left wheel forward by hand and watch which port's degree readout changes (`Port A`), then roll the right wheel (`Port E`).
3. **Set Pair Order:** Use `A+E` (`Left + Right`) in `set movement motors to [A+E v]`. If the robot drives backward on a forward command, swap to `E+A`.
4. **Attachment Motors:** Confirm your arms, claws, or racks are connected to **`Port B`** and **`Port C`**.

---

## Part 2: Movement Motor Calibration (`A+E`)

### Step A — Wheel Circumference (`wheel_circumference`)
Rubber tires compress slightly under robot weight, shifting the real distance traveled per rotation. Testing over **5 rotations** averages out small ruler measurement errors.

1. Place a piece of painter's tape on the mat and align the robot's front axle (or a fixed LEGO pointer) directly over the tape edge.
2. Run **`VerifyDistance`** with `speed = 30` and `rotations = 5`:
   ```text
   when program starts
   └── VerifyDistance (30) (5)
   ```
3. Mark where the same axle/pointer stops and measure the total distance in centimeters (for Robotina: **`138.0 cm`**).
4. Divide the measured distance by `5`:
   $$\text{wheel\_circumference} = \frac{138.0\text{ cm}}{5} = \mathbf{27.6\text{ cm}}$$
5. Update `set [wheel_circumference v] to [27.6]` inside your **`Setup`** MyBlock.

---

### Step B — Gyro Turn Calibration (`calibration_factor`)
Because a heavy robot has rotational momentum, telling the motors to stop at the exact instant the gyro reads `90°` causes the robot to coast past the target.

1. Align a straight edge of the robot's chassis flush against a straight line on the mat.
2. Run **`FindCalibrationFactor`** using a **`90°`** or **`180°`** test turn *(do not exceed `178°` because the SPIKE Prime yaw sensor wraps around at `±179°`)*:
   ```text
   when program starts
   └── FindCalibrationFactor (10) (175)
   ```
3. Check where the robot stops relative to the target angle:
   * **Overshoots past the line:** Lower `yaw_threshold` (e.g., from `175` to `173`).
   * **Stops short before the line:** Raise `yaw_threshold` (e.g., from `175` to `177`).
4. Once the robot lands squarely on the `90°` or `180°` mark, divide your winning `yaw_threshold` by the true target angle:
   $$\text{calibration\_factor} = \frac{\text{Tuned yaw\_threshold}}{\text{True Target Angle}}$$
   * **Robotina 90° test:** `87 ÷ 90 = 0.9667`
   * **Robotina 180° test:** `175 ÷ 180 = 0.9722`
   * **Average value:** **`0.9694`** (rounded to **`0.97`**)
5. Pass **`0.97`** as the `calibration_factor` input in **`GyroTurn`**.

---

### Step C — Tuning `Kp` (Proportional Steering Gain)
Inside **`GyroStraight`**, `Kp` controls how aggressively the robot corrects when `yaw angle` drifts away from `0°`.

1. Run `GyroStraight` across a long stretch of the mat (`80–100 cm` at `40%` speed) starting with `Kp = 1.5`:
   ```text
   when program starts
   ├── Setup
   └── GyroStraight (90) (40) (1.5)
   ```
2. Observe the robot's path and adjust `Kp` if needed:

| Observation During Run | Cause | Adjustment |
| :--- | :--- | :--- |
| Drives arrow-straight | Well-balanced gain | Keep **`Kp = 1.5`** |
| Wiggles / zig-zags left and right | Gain is too aggressive | Lower `Kp` to **`1.0 – 1.2`** |
| Slowly drifts off course without snapping back | Gain is too weak | Raise `Kp` to **`1.8 – 2.0`** |
| Spins in a circle immediately | Steering sign is inverted | Swap `+` and `-` in `start moving at` |

---

## Part 3: Attachment Motor Calibration (`B` & `C`)

Always program attachments using **degrees** rather than **seconds**. Degrees stay accurate regardless of battery charge, whereas time-based moves slow down as battery voltage drops.

### Step A — Measure Safe Degree Limits
1. Open the **Hub Dashboard** in the SPIKE Prime app.
2. Move the attachment by hand to its **starting "home" position** (how it sits in the launch area) and record the degree value for **Port B** or **Port C** (e.g., `120°`).
3. Slowly move the attachment by hand to its **full active/lowered position**, stopping just before it touches a LEGO frame beam or the mat, and record the new degree value (e.g., `265°`).
4. Subtract the two readings to find your total travel range:
   $$\text{Travel Degrees} = 265° - 120° = \mathbf{145°}$$
5. **Apply a Safety Buffer:** Subtract **`5°–10°`** (e.g., use **`135°–140°`**) if the mechanism approaches a physical hard stop so the motor never stalls or skips gears.

---

### Step B — Configure Hold Mode & Speed Limits
1. **Lock Positions at Stop:** Ensure your **`Setup`** MyBlock includes:
   * `[B v] set motor to [hold position v] at stop`
   * `[C v] set motor to [hold position v] at stop`
   *(This actively holds raised arms or loaded attachments steady against gravity while driving.)*
2. **Keep Speed Controlled:** Set attachment motor speeds between **`30%` and `50%`** (`[B v] set speed to [40] %`). Running heavy attachments at `100%` speed causes gear skipping and inconsistent stopping angles.
3. **Build a Physical Home Stop:** Add a LEGO beam or pin at the attachment's home position so technicians can push the arm firmly against the hard stop before every launch, guaranteeing an identical `0°` starting point.

---

## Quick Tournament Pre-Check Checklist

- [ ] **Battery Level:** Charge hub above `80%` before competition rounds.
- [ ] **Wheel Wipe:** Clean dust off drive tires (`A` and `E`) with a damp cloth before every match.
- [ ] **Attachment Zeroing:** Push `B` and `C` attachments firmly against their physical home stops in base.
- [ ] **Quick 90° Check:** Run one `GyroTurn (90) (30) (0.97)` on the practice table to verify floor friction matches your mat.
