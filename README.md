# Multi-Target Debugging with Multiple ST-LINK/V2 in STM32CubeIDE

This repository contains a guide to configuring and running simultaneous debug sessions for multiple STM32 microcontrollers on a single PC.

## What is this for?
When working on multi-board systems (e.g., interacting HMI, Main, and Power boards), debugging inter-device communication (SPI, I2C, CAN, UART) is complex. This setup allows you to:
- **Debug multiple microcontrollers simultaneously** within a single STM32CubeIDE window.
- **Step through code and hit breakpoints** across different targets in real time.
- **Analyze states and race conditions** without swapping USB cables or guessing the remote device's state.

---

## Documentation Guides

* 📘 **[Step-by-Step Tutorial](TUTORIAL_MULTI_DEBUG.md)**: Find ST-LINK serial numbers, configure GDB ports, and create a "Launch Group" for one-click startup.
* ❓ **[FAQ & Troubleshooting](FAQ_TROUBLESHOOTING.md)**: Solutions for port conflicts (`bind` errors), USB power limits, and sharing configuration files via Git.

---

## Key Benefits
* **Efficiency:** Drastically reduces time spent debugging multi-MCU communication.
* **Precision:** Synchronized stepping and breakpoint execution across different targets.
* **Zero Cost:** Uses existing ST-LINK/V2 probes and standard STM32CubeIDE features.
