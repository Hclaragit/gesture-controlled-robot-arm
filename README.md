# Gesture-Controlled Robotic Arm — Control Script

A Python script that lets a webcam hand-tracking view control a small 4-servo robotic arm over Bluetooth serial. It uses MediaPipe's pre-trained hand-landmark model through OpenCV, converts a few hand measurements into servo target angles, and sends position commands to an ESP32-based arm.

This was a group mini-project for ENGG1101 at HKU. This repository contains only the laptop-side Python script, not the whole project.

## My contribution

My part was the laptop-side software in `engg1101robot_final.py`:

- hand-landmark detection with OpenCV and MediaPipe
- mapping hand position and finger distance to target angles for four servos
- smoothing and deadzone filtering of the angles
- generating the Bluetooth serial command strings

The rest of the project was done by the team, including the physical arm and the ESP32 firmware that receives the commands. That firmware, the mechanical build, the camera-monitoring subsystem and the report are not part of this repository, and I am not claiming them as my work.

<!-- TODO(author): add teammates' names here only with their permission. -->

## What the script does

- Reads frames from the default webcam and tracks one hand with MediaPipe Hands
- Uses these measurements, all in normalised image coordinates:
  - horizontal offset of the middle-finger base from the wrist, which controls the base rotation
  - vertical wrist position, which controls shoulder and elbow
  - thumb-to-index fingertip distance, which controls the gripper
- Maps them to angles with linear mapping and clamping
- Smooths the angles with an exponential filter (`ALPHA`) and ignores changes smaller than a deadzone (`ANGLE_DEADZONE`)
- Converts angles to servo pulse values and sends `SET,<s1>,<s2>,<s3>,<s4>,<time_ms>` over serial, rate-limited and only when a value changes enough
- Checks the connection at start-up by sending `POS` and waiting for a reply containing `POS,`
- Moves the elbow back towards a home position when no hand is seen for a short time
- Shows the camera view with landmarks and current angles; press `q` to quit

## Tech stack

Python, OpenCV (`cv2`), MediaPipe (legacy `mp.solutions.hands` API), PySerial.

## Hardware and setup requirements

This script cannot do anything useful without the hardware:

- A webcam
- The ESP32-based robot arm, paired over Bluetooth, which accepts the `POS` and `SET,...` text commands. The firmware is not included here, so the protocol above is inferred from this script only.
- The serial port name is hard-coded as `/dev/cu.ENGG1101_K1` (macOS style) at 115200 baud. Edit `BT_PORT` for your machine.

```
pip install -r requirements.txt
python engg1101robot_final.py
```

The script uses `mp.solutions`, which newer MediaPipe releases may no longer provide. If that import fails, try an older MediaPipe release. No version has been pinned because none was verified here.

## Verification status

- Not run in this repository: it needs the arm and a Bluetooth connection, and the available virtual environment does not contain the dependencies.
- Static reading of the code only.

The team's course report describes tests with 10 trials per test and reports improved completion times and success rates after tuning. Those figures come from that report and were not re-measured here, and the sample is small.

## Limitations and known issues

- No custom gesture model was trained. Only MediaPipe's pre-trained hand landmarks are used, with hand-written threshold mappings.
- The thresholds are tuned by hand for one setup (camera position, lighting, arm) and have not been tested on other users or conditions.
- The script runs its main logic at module level, with no `main()` or `if __name__ == "__main__"` guard. Importing it opens the serial port.
- `is_fist()` and `palm_size` are defined but not used for control. The gripper is controlled by thumb-to-index distance, not by a fist.
- The shoulder target is clamped to 0–100, while the home angle is 105.
- On no-hand reset only the elbow returns to home.
- No safety layer: there is no emergency stop, workspace limit check or collision handling in this script.
- It is a classroom prototype and not meant for real construction or safety-critical use.

## Credits

Group mini-project for ENGG1101, HKU. Team names to be added with their permission.
