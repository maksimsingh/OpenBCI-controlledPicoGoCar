# OpenBCI-Controlled PicoGoCar

A real-time human-machine interface where EMG (muscle) signals are used to control a mobile robot. Signals are acquired with an OpenBCI Ganglion board, processed on a PC, and translated into motor commands sent to a PicoGo robot over USB.

**Control loop:** Human intent -> EMG signal -> Processing -> Commands -> Robot motion

The goal is reliable, low-latency detection of human intent and its immediate execution -- not autonomous navigation.

Demo: [Google Drive folder](https://drive.google.com/drive/folders/1jnBWf2pzapOHx9AfdqqYshw1g_xyr_ch?usp=drive_link)

## Architecture

```
User (muscle contraction)
        |
OpenBCI Ganglion
        |
Computer (BrainFlow + signal processing)
        |
      USB
        |
Raspberry Pi Pico (PicoGo)
        |
   Motors -> Movement
```

Signal processing runs on the PC for stability and low latency; the Pico only handles motor control.

## Hardware

- OpenBCI Ganglion + EMG electrodes
- PicoGo robot (Raspberry Pi Pico)
- USB connection
- PC running Python

## Software

- **BrainFlow** -- EMG signal acquisition
- **Python + NumPy** -- signal processing (RMS)
- **Serial (USB)** -- PC <-> Pico communication
- **MicroPython** -- robot motor control

## How It Works

1. **Calibration** -- the system measures 10s of baseline EMG activity per channel and computes dynamic thresholds from the mean and standard deviation.
2. **Detection** -- RMS of each channel is computed in real time; hysteresis (separate ON/OFF thresholds) prevents unstable switching between states.
3. **Control mapping:**

   | Action | Command |
   |---|---|
   | Left contraction | Left |
   | Right contraction | Right |
   | Chest contraction | Forward |
   | Relaxed | Stop |

4. **Communication** -- single-character commands are sent over serial: `F` forward, `L` left, `R` right, `S` stop.
5. **Execution** -- the Pico reads incoming commands and drives the motors via PWM.

## Key Decisions

- **USB over wireless** -- lower latency, higher reliability.
- **RMS-based detection** -- computationally cheap and stable for real-time use.
- **Hysteresis + debounce** -- separate activation/deactivation thresholds, plus minimum hold and switch dead-time, to prevent command flickering from noisy EMG signal.

## Safety

The system defaults to **STOP** whenever no valid signal is detected.

## Notes

- Slight drift during forward motion is expected (motor differences).
- Turning behavior may vary due to EMG signal variability between sessions.

## Future Improvements

- Proportional speed control (currently fixed speed per command)
- Adaptive thresholds (currently set once during calibration)
- Additional signal filtering
- More gesture inputs
- Wireless communication

## Summary

A closed-loop system where biological (EMG) signals directly and reliably control a robot in real time.
