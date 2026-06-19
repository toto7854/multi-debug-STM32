# FAQ & Troubleshooting: Multi-Target Debugging

This Q&A guide addresses common issues, edge cases, and best practices for debugging multiple microcontrollers simultaneously on a single computer.

---

## 1. GDB Server and Connection Errors

### Q: I get an error: "Could not start GDB server" or "Address already in use: bind". How do I fix it?
* **Cause:** Two debug configurations are trying to use the same GDB Server port (typically the default `61234`) or SWO port (`61235`).
* **Solution:** 
  1. Open **Debug Configurations...** for your second/third project.
  2. Navigate to the **Debugger** tab.
  3. Change the **GDB Server Port** to a unique value (e.g., `61236`).
  4. Change the **SWO Port** to a unique value (e.g., `61237`).
  5. Apply the changes and restart the launch group.

### Q: STM32CubeIDE displays "Failed to connect to ST-LINK" or "ST-Link USB communication error".
* **Cause A:** The selected ST-LINK serial number is not plugged in, or the USB port went to sleep.
  * **Fix:** Click **Scan** in the Debugger settings to verify if the serial number is detected. Try unplugging and re-plugging the probe.
* **Cause B:** Power starvation. Multiple boards and debuggers can draw more current than a single USB port can supply (typically 500mA for USB 2.0).
  * **Fix:** Always use a **powered USB hub** (with its own external wall-power adapter) when debugging more than two boards. Avoid running multiple boards off laptop battery power.

---

## 2. Git and Team Collaboration Best Practices

### Q: How do we share project configurations in Git without breaking things for other developers who have different physical ST-LINK probes?
* **Problem:** Debug configurations in Eclipse/STM32CubeIDE are stored in `.launch` files. If you save them in the project and commit them, they will contain your specific ST-LINK Serial Number (S/N). When a teammate pulls the code, their debug sessions will fail because they don't have your specific probe.
* **Best Practice Solution:**
  1. **Do not commit local `.launch` files:** By default, Eclipse stores launch configurations in the local workspace directory (which is not committed to Git). Keep it this way if possible.
  2. **Template Configurations:** If you want to share debug configurations, save them in the project directory but **leave the ST-LINK S/N field blank** in the committed version.
     * When a developer launches it the first time, STM32CubeIDE will prompt: *"Multiple ST-LINKs found, please select one."*
     * Once they select their local probe, they can save it locally.
  3. Add local/modified `.launch` files to your `.gitignore`:
     ```gitignore
     # Ignore developer-specific launch configurations
     *.launch
     ```

---

## 3. Advanced Q&A

### Q: Can we debug different STM32 microcontroller families simultaneously (e.g., an STM32F4 on one board and an STM32L4 on another)?
* **Yes, absolutely.** The ST-LINK GDB server dynamically detects the target core architecture (Cortex-M0+, M4, M7, etc.) and uses the appropriate registers. You just need to ensure that each project's debug configuration is pointed to the correct binary and target hardware.

### Q: Does terminating or restarting one debug session affect the others?
* **No.** Each session runs in its own GDB server instance on a distinct network port. You can stop, compile, flash, and restart `Project_IHM` without interrupting `Project_Principal`.
* > [!TIP]
  > This independence is highly valuable for testing **robustness**: you can halt/suspend one microcontroller (simulating a crash or communication timeout) and observe how the other microcontroller's state machine handles the communication loss.

### Q: My debugger is laggy, or Live Expressions take a long time to update.
* **Cause:** High GDB server overhead when monitoring many variables or trace lines simultaneously.
* **Fixes:**
  1. **Increase the Refresh Interval:** Right-click inside the **Live Expressions** tab, select *Refresh Interval...*, and increase it from the default `200ms` to `500ms` or `1000ms`.
  2. **Limit Expressions:** Remove unused variables from the Live Expressions view.
  3. **Disable SWO Trace:** If you are not actively using Serial Wire Output (SWO) tracing (e.g., `printf` redirection), disable it in the **Debugger** tab of your debug configuration to free up bandwidth.

### Q: How does resetting work? If I click the "Restart" button, does it reset both boards?
* **No.** Clicking the **Restart** (yellow play/pause arrow) or **Reset** button in the debug control panel only resets the microcontroller associated with the *currently selected thread* in the **Debug** view.
* To reset all boards, select each thread in the Debug view one-by-one and click the Reset button.
