# T-Rexx Autonomous RC Platform

This folder tracks the integration work for converting legacy R/C cars and trucks into autonomous Raspberry Pi vehicles with voice commands, hand gestures, person-following, mapping, and waypoint navigation.

## Stage 1 goal

Get one Raspberry Pi 5 vehicle safely driving under software control with a camera before adding voice, gestures, SLAM, or Nav2.

## Core stack

- DonkeyCar: vehicle loop, camera, calibration, telemetry, steering/throttle control, initial autopilot
- MediaPipe + hand-gesture-recognition: gesture and person input
- openWakeWord + Vosk: fast offline wake phrase and command vocabulary
- whisper.cpp: richer offline speech commands on Pi 5
- ROS 2 Nav2 + SLAM Toolbox: later mapping and goal navigation
- F1TENTH gym/labs: simulation and advanced driving algorithms

## Integration rule

Voice, gestures, autonomous driving, and future ROS 2 navigation do not command motors directly. They emit high-level commands into a shared command/safety manager. The safety manager decides whether motion is allowed, applies speed limits, and can always force throttle to zero.

## Stage sequence

1. Inventory one R/C chassis and identify its steering/throttle hardware.
2. Install Raspberry Pi OS and DonkeyCar on the Pi 5.
3. Bring up the camera and record frames.
4. Connect steering and throttle through the appropriate interface.
5. Calibrate steering endpoints and ESC neutral/forward/reverse at low speed.
6. Add a physical kill switch and software watchdog.
7. Record manually driven training data and validate basic DonkeyCar autonomy.
8. Add gesture input.
9. Add wake-word + offline voice input.
10. Add person-following and obstacle sensors.
11. Add ROS 2, SLAM Toolbox, and Nav2 for mapping/waypoints.

## Related forks

- tk990104/donkeycar
- tk990104/mediapipe
- tk990104/hand-gesture-recognition-using-mediapipe
- tk990104/vosk-api
- tk990104/whisper.cpp
- tk990104/openWakeWord
- tk990104/navigation2
- tk990104/slam_toolbox
- tk990104/f1tenth_gym_ros
- tk990104/f1tenth_labs_openrepo

See `STAGE1_HARDWARE.md`, `STAGE1_PI5_SETUP.md`, and `ARCHITECTURE.md` in this folder.
