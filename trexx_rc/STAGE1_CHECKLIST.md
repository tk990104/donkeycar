# Stage 1 First Vehicle Bring-Up Checklist

## Goal

Bring one Raspberry Pi 5 R/C vehicle to safe, repeatable software-controlled steering/throttle with camera capture before adding voice, gestures, ROS 2, or autonomous navigation.

## Hardware identification

- [ ] Choose first R/C car/truck chassis
- [ ] Identify hobby-grade servo + ESC vs toy-grade DC steering/drive motors
- [ ] Record motor type: brushed or brushless
- [ ] Record battery chemistry, voltage, and connector
- [ ] Record ESC/BEC rating if present
- [ ] Measure mounting area for Pi/electronics deck

## Raspberry Pi 5

- [ ] Flash/update Raspberry Pi OS
- [ ] Enable SSH and I2C
- [ ] Verify stable regulated Pi power
- [ ] Install current DonkeyCar environment
- [ ] Verify Pi Camera

## Drivetrain

- [ ] Wire PCA9685 if hobby-grade servo + ESC
- [ ] Or select an H-bridge interface if toy-grade motors
- [ ] Verify common control ground
- [ ] Install physical traction kill switch
- [ ] Test steering with driven wheels off the ground
- [ ] Calibrate steering left/right endpoints
- [ ] Calibrate ESC stopped/forward/reverse
- [ ] Cap Stage 1 throttle to low speed

## Safety

- [ ] Verify physical kill switch stops traction power
- [ ] Add command timeout/watchdog that forces neutral throttle
- [ ] Verify boot state cannot accidentally apply throttle
- [ ] Verify loss of controlling process returns to neutral

## Data and autonomy

- [ ] Record camera + steering/throttle telemetry
- [ ] Complete first controlled manual laps
- [ ] Train/test first DonkeyCar autopilot only after safe manual control is reliable

## Later stages

- [ ] MediaPipe gestures/person tracking
- [ ] openWakeWord + Vosk
- [ ] whisper.cpp
- [ ] Obstacle sensors
- [ ] ROS 2 + SLAM Toolbox + Nav2
- [ ] Multi-vehicle coordination
