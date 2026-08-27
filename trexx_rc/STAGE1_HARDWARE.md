# Stage 1 Hardware Plan

## Preferred first-vehicle architecture

Use the Raspberry Pi 5 as the main computer. For a hobby-grade R/C chassis with a steering servo and electronic speed controller (ESC), use DonkeyCar's `PWM_STEERING_THROTTLE` drivetrain. A PCA9685 I2C PWM board is the preferred first interface because it provides stable servo-style PWM without making Linux GPIO timing responsible for steering and throttle.

### Signal path

Pi 5 -> I2C -> PCA9685 -> steering servo signal + ESC signal

Pi Camera -> CSI camera connector -> Pi 5

R/C traction battery -> ESC -> motor

Dedicated 5V regulator/UBEC -> Raspberry Pi 5

The Pi and the control electronics must share a common ground with the ESC signal ground.

## Power rules

- Do not power the traction motor from the Raspberry Pi.
- Do not assume the Pi 5V rail is appropriate for a high-current steering servo.
- Prefer a dedicated regulated supply for the Pi 5 sized for the Pi and attached USB/camera hardware.
- Servo power can come from a suitable ESC BEC or separate regulator, but voltage must match the servo rating.
- Join grounds between the Pi/PCA9685 control side and the servo/ESC signal side.
- Add an inline physical kill switch that can disable traction power independently of software.
- Initial calibration should be performed with the driven wheels raised off the ground.

## Hobby-grade chassis wiring

| Connection | From | To |
|---|---|---|
| I2C SDA | Pi 5 GPIO2 / pin 3 | PCA9685 SDA |
| I2C SCL | Pi 5 GPIO3 / pin 5 | PCA9685 SCL |
| Logic power | Pi 5 3.3V | PCA9685 VCC if board supports 3.3V logic |
| Ground | Pi ground | PCA9685 GND and ESC/servo signal ground |
| Steering signal | PCA9685 channel 1 | Steering servo signal |
| Throttle signal | PCA9685 channel 0 | ESC receiver/signal input |
| Servo rail | BEC/regulator | PCA9685 V+ / servo power rail |
| Pi power | Dedicated 5V regulator/UBEC | Pi 5 USB-C or supported 5V input |
| Camera | Pi 5 CSI connector | Raspberry Pi Camera |

## DonkeyCar starting configuration

Use these as placeholders only; every chassis must be calibrated before driving.

```python
CAMERA_TYPE = "PICAM"
IMAGE_W = 160
IMAGE_H = 120
CAMERA_FRAMERATE = 20

DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"
PWM_STEERING_THROTTLE = {
    "PWM_STEERING_PIN": "PCA9685.1:40.1",
    "PWM_STEERING_SCALE": 1.0,
    "PWM_STEERING_INVERTED": False,
    "PWM_THROTTLE_PIN": "PCA9685.1:40.0",
    "PWM_THROTTLE_SCALE": 1.0,
    "PWM_THROTTLE_INVERTED": False,
    "STEERING_LEFT_PWM": 460,
    "STEERING_RIGHT_PWM": 290,
    "THROTTLE_FORWARD_PWM": 500,
    "THROTTLE_STOPPED_PWM": 370,
    "THROTTLE_REVERSE_PWM": 220,
}
```

Do not copy the endpoint PWM numbers into a live vehicle without calibration. They are DonkeyCar example values, not universal values.

## If the old R/C vehicle is toy-grade

Some older toy R/C cars do not have a standard three-wire steering servo and hobby ESC. They may use a brushed DC motor for steering and another DC motor for propulsion on a combined receiver board. In that case, bypass or replace the original receiver electronics and use an H-bridge motor driver. DonkeyCar already exposes drivetrain modes for servo + H-bridge, DC steering + throttle, and differential/two-wheel drive.

Before connecting any chassis, record:

| Item | What to identify |
|---|---|
| Steering | 3-wire hobby servo, DC steering motor, or mechanical actuator |
| Throttle | Hobby ESC, brushed motor board, or combined receiver/ESC |
| Motor | Brushed or brushless |
| Battery | Chemistry, nominal voltage, connector |
| ESC/BEC | Voltage/current rating and whether it provides receiver power |
| Receiver connections | Standard servo plugs or proprietary PCB wiring |
| Chassis space | Flat area for Pi, regulators, camera mast, and sensors |

## Stage 1 sensor set

Start with only the camera and a physical kill switch. Add obstacle sensors after basic steering/throttle and camera operation are reliable. The first obstacle layer can use short-range ToF/ultrasonic sensors; LiDAR belongs in the later SLAM/Nav2 phase.
