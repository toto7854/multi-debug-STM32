# Step-by-Step Tutorial: Multi-Target Debugging in STM32CubeIDE

This guide walks you through the configuration required to debug multiple microcontrollers simultaneously using multiple ST-LINK/V2 probes connected to a single PC.

---

## Prerequisites
* **STM32CubeIDE** (v1.10.0 or later recommended).
* **STM32CubeProgrammer** installed (integrated with STM32CubeIDE, but standalone is helpful).
* Multiple **ST-LINK/V2 probes** (or onboard ST-LINK debuggers on Nucleo/Discovery boards).
* Multiple target boards (e.g., your IHM, UC Principal, and UC Power boards).
* **Micro-USB / Mini-USB cables** for each probe.

---

## Step 1: Physical Setup & Probe Labeling

To avoid confusion during debugging, you must physically identify which ST-LINK probe is connected to which microcontroller board.

1. Connect each ST-LINK probe to its corresponding target board.
2. Label each physical ST-LINK probe with a label maker or tape (e.g., `"Probe A - IHM"`, `"Probe B - UC Principal"`).
3. Connect all probes to your PC. 
   > [!TIP]
   > If you run out of USB ports, use a **powered USB 3.0 Hub**. Unpowered hubs may fail to deliver enough current to multiple ST-LINK probes and target microcontrollers simultaneously.

---

## Step 2: Retrieve Probe Serial Numbers

Each ST-LINK probe has a unique hardware Serial Number (SN). We need this ID to instruct STM32CubeIDE which GDB server controls which physical probe.

### Method A: Using STM32CubeProgrammer GUI (Easiest)
1. Open **STM32CubeProgrammer**.
2. On the right-hand panel, set the Connection type to **ST-LINK**.
3. Locate the **Serial number** dropdown field.
4. Click the **Refresh/Scan** button (circular arrows).
5. All connected ST-LINK probes will appear in the dropdown list.
6. Copy the serial numbers and associate them with your labels (e.g., `066EFF535051717867152438` is **Probe A - IHM**).

### Method B: Using STM32CubeProgrammer CLI (Fastest for script lovers)
Open a command prompt (or PowerShell) and navigate to your STM32CubeProgrammer installation, then run:
```powershell
# List all connected ST-LINK probes
& "C:\ST\STM32CubeProgrammer\bin\STM32_Programmer_CLI.exe" --list
```
This will print a list of all detected probes with their index and **Serial Number**.

---

## Step 3: Configure Debug Configurations in STM32CubeIDE

You must configure a separate Debug Configuration for each project. Here is how to configure them so they do not conflict.

Let's assume we have two projects: **Project_IHM** and **Project_Principal**.

### Configuration for Project 1 (Project_IHM)
1. In STM32CubeIDE, right-click **Project_IHM** -> **Debug As** -> **Debug Configurations...**
2. Select **STM32 Cortex-M C/C++ Application** and click the **New Launch Configuration** icon.
3. Name it: `Project_IHM_Debug`.
4. Go to the **Debugger** tab:
   * **Debug Probe:** Select `ST-LINK (ST-LINK GDB Server)`.
   * Under **Interface**: Select your interface (usually `SWD`).
   * Under **Connection Settings**:
     * Check **Shared ST-LINK**.
     * Look for **ST-LINK S/N** and click the **Scan** button.
     * Select the Serial Number that corresponds to **Probe A - IHM** from the dropdown list.
     * **GDB Server Port:** Leave as default `61234`.
     * **SWO Port:** Leave as default `61235`.
5. Click **Apply**.

---

### Configuration for Project 2 (Project_Principal)
1. In the Debug Configurations window, select **STM32 Cortex-M C/C++ Application** -> Create another new launch configuration.
2. Name it: `Project_Principal_Debug`.
3. Go to the **Debugger** tab:
   * **Debug Probe:** Select `ST-LINK (ST-LINK GDB Server)`.
   * Under **Interface**: Select `SWD`.
   * Under **Connection Settings**:
     * Check **Shared ST-LINK**.
     * Click **Scan** next to **ST-LINK S/N**.
     * Select the Serial Number corresponding to **Probe B - UC Principal**.
     * > [!IMPORTANT]
       > **GDB Server Port:** Change this to **`61236`** (Incremented to avoid conflict).
       > **SWO Port:** Change this to **`61237`** (Incremented to avoid conflict).
4. Click **Apply**.

---

### Port Allocation Guide for Multiple Targets

If you have more than two boards, continue incrementing the GDB and SWO ports:

| Target Name | ST-LINK S/N Selection | GDB Port | SWO Port |
| :--- | :--- | :--- | :--- |
| Target 1 (IHM) | `Probe A Serial Number` | **61234** | **61235** |
| Target 2 (Principal) | `Probe B Serial Number` | **61236** | **61237** |
| Target 3 (Power) | `Probe C Serial Number` | **61238** | **61239** |

---

## Step 4: Group Launch (One-Click Multi-Debugging)

Instead of launching each debug configuration manually one-by-one, you can group them together to start them in sequence automatically.

1. In STM32CubeIDE, go to **Debug Configurations...**
2. Double-click **Launch Group** (at the bottom of the left list) to create a new group.
3. Name it: `System_Multi_Debug_Group`.
4. Click the **Add...** button on the right:
   * Select `Project_IHM_Debug`.
   * **Action:** Select `Launch`.
   * **Post-launch action:** Select `Wait for console output` or `Delay` (e.g., `2` seconds) to let the GDB server start up before launching the next one.
5. Click the **Add...** button again:
   * Select `Project_Principal_Debug`.
   * **Action:** Select `Launch`.
6. Click **Apply** and **Debug**.

---

## Step 5: Managing the Multi-Debug Session

Once launched, STM32CubeIDE will enter the **Debug Perspective**. Here is how to navigate it:

### 1. The Debug View (Call Stack)
This panel lists all active sessions. You can expand them to see the call stack of each microcontroller:
* `Project_IHM_Debug [STM32 Cortex-M C/C++ Application]` -> Thread #1 (Running/Suspended)
* `Project_Principal_Debug [STM32 Cortex-M C/C++ Application]` -> Thread #1 (Running/Suspended)

*Select* a thread in this view to update the editor window, the **Variables** view, and the **Live Expressions** view to match that specific microcontroller.

### 2. Switching Consoles
Under the **Console** tab, you will see output from all GDB servers. Use the **Display Selected Console** drop-down icon (a blue monitor icon) to switch between the logs of the IHM debugger and the Principal debugger.

### 3. Controlling Execution
You can control the microcontrollers independently or together:
* **Suspend/Resume Individual Target:** Click on the target in the *Debug View* first, then click **Resume** (F8) or **Suspend**.
* **Global Breakpoints:** If you place a breakpoint in `ihm.cpp`, it will only trigger when the IHM board hits that line. A breakpoint in `principal.cpp` will only trigger when the Principal board hits it.

---

*Move on to the **[FAQ & Troubleshooting Guide](FAQ_TROUBLESHOOTING.md)** for handling common issues during multi-debug sessions.*
