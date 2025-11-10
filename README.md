# Zephyr-Alif (Work in Progress)
Zephyr examples for an Alif Semiconductor DK-E7 board. The debug launch.json file has been adapted to load the ELF image built by the Zephyr West tool debugged by the [CMSIS-Debugger](https://marketplace.visualstudio.com/items?itemName=Arm.vscode-cmsis-debugger) extension.

[**Examples/IPM_ARM_MHUv2**](https://github.com/Open-CMSIS-Pack/Zephyr-Alif/tree/main/Examples/IPM_ARM_MHUv2) contains a [IPM_ARM_MHUv2](https://github.com/alifsemi/sdk-alif/tree/v1.3.2/samples/drivers/ipm/ipm_arm_mhuv2) CMSIS-solution-based project from [Alif-Zephyr-SDK](https://github.com/alifsemi/sdk-alif). This example demonstrates the dual-core debugging and inter-core communication between two Cortex-M55 cores using Arm's Message Handling Unit v2 (MHUv2) . Also, it showcases the use of Zephyr's Inter-Processor Mailbox (IPM) API to exchange messages and trigger interrupts between cores.

[**Examples/LPI2C**](https://github.com/Open-CMSIS-Pack/Zephyr-Alif/tree/main/Examples/LPI2C) contains a [LPI2C](https://github.com/alifsemi/sdk-alif/tree/v1.3.2/samples/drivers/lpi2c) CMSIS-solution-based project from [Alif-Zephyr-SDK](https://github.com/alifsemi/sdk-alif). LPI2C (Low Power Inter-Integrated Circuit) is a low-power version of the standard I2C bus controller.
This example uses two threads to emulate an I2C master and slave communicating through the LPI2C driver.
All data transfers occur internally via hardware loopback.

# Steps to setup the required tools for Zephyr
1. Install dependencies to your PC:
   - python3
   - python3-pip
2. Install west and pyelftools with this command:
   ```bash
   python3 -m pip install west pyelftools jsonschema
   ``` 
3. Verify the west installation by running:
   ```bash
   west --version
   ```
   - In case you see the error `west: command not found`, add the west.exe in PATH if reqired

More details can be found in [Zephyr Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html).

# SETOOLS

Before flashing this example on the AppKit E7 board it is required to program the ATOC of the device using the Alif SETOOLS. This process only has to be done once for each single- or dual-core project.

Refer to the section [Usage](https://github.com/alifsemi/alif_ensemble-cmsis-dfp/blob/main/docs/Overview.md#usage)
in the overview page of the Alif Semiconductor Ensemble DFP/BSP for information on how
to setup these tools.

In VS Code use the menu command **"Terminal - Run Tasks"** and execute:

- "Alif: Install M55_HE and M55_HP debug stubs (dual core configuration)"

# Steps to build and debug the Zephyr example
1. Open the folder *./Zephyr-Alif* in VS code.

2. In Terminal, initialize and update the Zephyr SDK-Alif workspace, if none exists:
   ```bash
   mkdir -p zephyrproject
   cd ./zephyrproject
   west init -m https://github.com/alifsemi/sdk-alif.git --mr v1.3.2
   west update 
   ```

3. To use the west tools for projects located in different folders, set the `ZEPHYR_BASE` environment variable to the *./Zephyr-Alif/zephyrproject/zephyr* folder:

   **For Ubuntu:**

   ```bash
   (echo; echo 'export ZEPHYR_BASE="/home/.../zephyrproject/zephyr"') >> ~/.bashrc
   source ~/.bashrc
   ```
   - Make sure to restart the VS Code.

   **For MacOS:**

   ```bash
   (echo; echo 'export ZEPHYR_BASE="/usr/.../zephyrproject/zephyr"') >> ~/.zshrc
   source ~/.zshrc
   ```
   - Make sure to fully quit VS Code, not just close the window. Otherwise, the changes won’t be applied.

   **For Windows:**
      - Set ```ZEPHYR_BASE``` to ```C:/.../Zephyr-Workspace/zephyr``` in Environment Variables, and restart the VS Code.

4. Press the **"Build solution"** button to build the example.

5. Start the CMSIS Debugger for dual-core debugging:
   - Start the **M55_HE CMSIS_DAP@pyOCD (launch)** debug session first, followed by **M55_HP CMSIS_DAP@pyOCD (attach)**.
   - After starting the second debug session, the program will halt at `cpu_idle.S`. This occurs because the second core still remains in its idle loop until a valid entry point is reached. To resolve this, add the following command under the **M55_HP CMSIS_DAP@pyOCD (attach)** section in the launch.json file:
   ```bash
   "initCommands": [
      "monitor reset halt",
      "tbreak main"
   ]
   ```
   - And remember to set `updateConfiguration:` to `manual` in order to prevent your settings from being overwritten.
