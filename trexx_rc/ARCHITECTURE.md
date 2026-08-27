# T-Rexx Autonomous RC Architecture

## Principle

All control sources produce intents. Only the safety/control arbiter is allowed to produce final steering and throttle commands.

```text
Camera ──────────────> DonkeyCar vision/autopilot ──┐
Camera ──────────────> MediaPipe gestures ──────────┤
Microphone -> WakeWord -> Vosk/Whisper ─────────────┤
ROS 2 Nav2 / waypoints ─────────────────────────────┤
Manual test controller ─────────────────────────────┤
                                                    v
                                            Command Arbiter
                                                    |
                                  ┌─────────────────┼─────────────────┐
                                  |                 |                 |
                            speed limits      obstacle gate      watchdog
                                  |                 |                 |
                                  └─────────────────┼─────────────────┘
                                                    v
                                           DonkeyCar drivetrain
                                                    |
                                          steering servo + ESC
```

## Command contract

Every high-level input should eventually normalize to a shared command object rather than manipulating PWM directly.

Example conceptual command:

```python
{
    "source": "voice",
    "mode": "follow_me",
    "steering": None,
    "throttle": None,
    "target": "person",
    "priority": 40,
    "expires_ms": 1000,
}
```

Direct motion commands can use normalized steering/throttle in the range -1.0 to +1.0, but PWM conversion remains inside the drivetrain layer.

## Priority model

Highest priority wins, but safety can always override all other sources.

| Priority | Source | Example |
|---|---|---|
| 100 | Emergency stop | physical/software E-stop |
| 90 | Collision safety | obstacle too close |
| 80 | Watchdog | command source timed out |
| 60 | Manual test control | technician driving during setup |
| 50 | Voice/gesture immediate command | stop, turn, come here |
| 40 | Nav2 / mission command | go home, patrol waypoint |
| 30 | Follow-me controller | track selected person |
| 20 | DonkeyCar autopilot | learned lane/path driving |

A voice or gesture `STOP` command should be promoted to emergency-stop priority rather than treated as an ordinary voice command.

## Operating modes

- SAFE_IDLE: steering centered, throttle neutral
- MANUAL_TEST: controlled low-speed setup/testing
- AUTOPILOT: DonkeyCar model drives
- FOLLOW_ME: person tracker supplies target direction/distance
- VOICE_COMMAND: command parsed into a mission or immediate action
- GESTURE_COMMAND: hand gesture parsed into mission/immediate action
- WAYPOINT_NAV: ROS 2/Nav2 goal navigation
- PATROL: sequence of waypoint goals
- RETURN_HOME: navigate to saved home/parking goal
- ESTOP: throttle neutral/disabled until explicitly reset

## Stage 1 implementation boundary

Stage 1 implements only SAFE_IDLE, MANUAL_TEST, camera capture, drivetrain calibration, physical kill switch, and a basic software timeout. Later stages add the other modes without changing the underlying motor wiring.

## Expansion interfaces

### Voice

openWakeWord -> Vosk for a small deterministic command vocabulary. whisper.cpp can be added for richer phrases after basic commands are reliable.

### Gestures

MediaPipe detects hands/person pose. The gesture-recognition fork maps landmarks to discrete commands. Gesture code emits intents to the arbiter; it never writes PWM.

### ROS 2 navigation

Nav2 and SLAM Toolbox are later-stage services. Their velocity/path output must be adapted into the same arbiter rather than bypassing it.

### Multi-vehicle future

Each vehicle should have its own identity, local safety controller, and local E-stop. Convoy/fleet coordination sends mission goals; it should never depend on a central computer for immediate collision stopping.
