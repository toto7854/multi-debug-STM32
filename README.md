# Next Practice: Multi-Target Debugging with Multiple ST-LINK/V2 in STM32CubeIDE

Welcome to the IDhall / Improve sharing portal for **Multi-Target Debugging**. This documentation is designed to establish a new "next practice" for our development team, enabling engineers to debug multiple STM32 microcontrollers simultaneously using multiple ST-LINK/V2 probes on a single PC.

## Why This Matters (Continuous Improvement)

In complex multi-board setups (such as systems featuring interacting **HMI/IHM**, **Main/Principal**, and **Power** boards), debugging communication protocols (SPI, I2C, CAN, UART) between microcontrollers has historically been a major bottleneck. 

Traditionally, developers would:
1. Debug one board at a time, guessing the state of the other.
2. Use print statements over UART, which changes timing and can hide race conditions.
3. Switch USB cables back and forth.

**The Next Practice:** By configuring STM32CubeIDE to target specific ST-LINK/V2 probes by their unique hardware Serial Numbers (SN) and assigning dedicated GDB ports, we can run multiple active debug sessions side-by-side in a single IDE window. We can inspect variables, set breakpoints, and trace execution flow across boards in real-time.

---

## Documentation Structure

Click on the links below to access the full guides:

| Document | Description | Key Focus Areas |
| :--- | :--- | :--- |
| 📘 **[Step-by-Step Tutorial](TUTORIAL_MULTI_DEBUG.md)** | Core configuration guide to set up multiple boards. | Serial Number detection, Debug configuration, GDB port offset. |
| ❓ **[FAQ & Troubleshooting](FAQ_TROUBLESHOOTING.md)** | Common errors, hardware recommendations, and QA. | Port conflicts, USB Hub bandwidth, GDB server lockups, team sharing. |

---

## Quick Benefits Matrix

* **Time Saved:** Up to 70% reduction in time-to-debug multi-MCU communication bugs.
* **Accuracy:** Real-time synchronization of breakpoints across multiple targets.
* **Cost:** Zero. Uses our existing ST-LINK/V2 probes and STM32CubeIDE.
* **Ease of Use:** Once configured, starting a multi-debug session is a single click.

---

*Submitted to IDhall / Improve for team sharing & process optimization.*
