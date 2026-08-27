# Raspberry Pi 5 Stage 1 Setup

## Target platform

- Raspberry Pi 5
- Current Raspberry Pi OS
- Raspberry Pi Camera via CSI
- DonkeyCar fork: `tk990104/donkeycar`
- Working branch: `trexx-autonomous-rc-stage1`

## Base OS setup

1. Flash Raspberry Pi OS with Raspberry Pi Imager.
2. Configure Wi-Fi and SSH during imaging if possible.
3. Boot the Pi and update packages.
4. Enable I2C for the PCA9685.
5. Verify the camera before installing the rest of the vehicle stack.

Suggested commands:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo raspi-config
```

In `raspi-config`, enable I2C and SSH. Current Pi camera support uses libcamera/picamera2 rather than the legacy camera stack.

## DonkeyCar environment

The current DonkeyCar branch uses `uv` for environment management. Raspberry Pi camera support depends on Debian system packages, so install those first and create the environment with system site packages visible.

```bash
sudo apt install -y git python3-libcamera python3-picamera2 i2c-tools
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Clone your fork:

```bash
cd ~
git clone https://github.com/tk990104/donkeycar.git
cd donkeycar
git checkout trexx-autonomous-rc-stage1
```

Create the Pi virtual environment using the Raspberry Pi OS system Python. On Raspberry Pi OS Trixie, DonkeyCar currently documents Python 3.13:

```bash
uv venv ~/env --python 3.13 --system-site-packages
source ~/env/bin/activate
uv pip install -e ".[pi,dev]"
```

Optionally activate the environment automatically for your shell.

## Verify camera

Before connecting motors, verify that the Pi sees the camera and that Picamera2 works. Do not troubleshoot camera and motor control simultaneously.

## Verify I2C

After the PCA9685 is wired, run:

```bash
i2cdetect -y 1
```

A standard PCA9685 normally appears at address `0x40`. Do not proceed to servo/ESC calibration until the board is detected consistently.

## Create a car application

Use the current DonkeyCar application/template workflow to create the local car directory. Keep vehicle-specific calibration in the car's `myconfig.py`; do not hard-code chassis-specific PWM values into the DonkeyCar library fork.

Recommended local layout:

```text
~/mycar/
├── manage.py
├── config.py
├── myconfig.py
├── data/
└── models/
```

## First-boot validation order

| Order | Test | Pass condition |
|---|---|---|
| 1 | Pi power | Stable boot; no undervoltage warnings |
| 2 | SSH/Wi-Fi | Reliable remote login |
| 3 | Camera | Live frames visible |
| 4 | I2C | PCA9685 detected at expected address |
| 5 | Steering | Servo moves within safe range with wheels raised |
| 6 | ESC neutral | Motor stays stopped at neutral command |
| 7 | Low forward | Wheels rotate slowly in expected direction |
| 8 | Low reverse | Reverse works only after ESC-required neutral/brake sequence |
| 9 | Kill switch | Traction power stops immediately |
| 10 | Software watchdog | Loss of command forces throttle to zero |

## Initial software speed policy

For Stage 1, cap commanded throttle aggressively. The goal is reliable steering, stopping, camera capture, and data recording—not speed. Raise limits only after the kill switch, neutral calibration, and watchdog have been tested repeatedly.

## What is intentionally not installed yet

Do not put MediaPipe, Whisper, Vosk, ROS 2, Nav2, and SLAM into the first vehicle image on day one. Bring up DonkeyCar + camera + drivetrain first. Add the other subsystems one at a time after the basic vehicle is stable.
