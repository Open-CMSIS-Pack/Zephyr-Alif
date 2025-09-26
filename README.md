# Zephyr-Alif
Zephyr examples for an Alif Semiconductor DK-E7 board. The debug launch.json file has been adapted to load the ELF image built by the Zephyr West tool debugged by the [CMSIS-Debugger](https://marketplace.visualstudio.com/items?itemName=Arm.vscode-cmsis-debugger) extension.

[**Examples/ipm_arm_mhuv2**](https://github.com/Open-CMSIS-Pack/Zephyr-Alif/tree/main/Examples/ipm_arm_mhuv2) contains a [ipm_arm_mhuv2](https://github.com/alifsemi/sdk-alif/tree/main/samples/drivers/ipm/ipm_arm_mhuv2) CMSIS solution example project from [Alif-Zephyr-SDK](https://github.com/alifsemi/sdk-alif), which is built by Zephyr West tool and debugged by CMSIS-Debugger extension. This example demonstrates the dual-core debugging in Keil Studio and inter-core communication between two Cortex-M55 cores using Arm's Message Handling Unit v2 (MHUv2) . Also, it showcases the use of Zephyr's Inter-Processor Mailbox (IPM) API to exchange messages and trigger interrupts between cores.

[**Examples/ipm_arm_mhuv2_sysbuild**](https://github.com/Open-CMSIS-Pack/Zephyr-Alif/tree/main/Examples/ipm_arm_mhuv2_sysbuild) is an example extended from [ipm_arm_mhuv2](https://github.com/alifsemi/sdk-alif/tree/main/samples/drivers/ipm/ipm_arm_mhuv2) with Zephyr [System Build](https://docs.zephyrproject.org/latest/build/sysbuild/index). For more information about System Build, please refer to the README.md file in the example.

# Steps to setup Zephyr environment
1. Install dependencies to your PC:
   - python3
   - python3-pip
   - wget
   - cmake
   - ninja
2. Install west and pyelftools with this command:
   ```bash
   python3 -m pip install west pyelftools
   ``` 
3. Make sure to add the executable `west.exe` into the **PATH** environment variable.
4. To reuse the Arm GNU compiler toolchain installed by vcpkg in Keil Studio VS Code, add system environment variables:
   - Set ```ZEPHYR_TOOLCHAIN_VARIANT``` to ```gnuarmemb```
   - Set ```GNUARMEMB_TOOLCHAIN_PATH``` to ```\Users\<UserName>\.vcpkg\artifacts\...\compilers.arm.arm.none.eabi.gcc\14.3.1```
   
   Similar for Arm Compiler 6:
   - Set ```ZEPHYR_TOOLCHAIN_VARIANT``` to ```armclang```
   - Set ```ARMCLANG_TOOLCHAIN_PATH``` to ```\Users\<UserName>\.vcpkg\artifacts\...\compilers.arm.armclang\6.24.0```

More details can be found in [Zephzr Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html).

# SETOOLS

Before flashing this example on the AppKit E7 board it is required to program the ATOC of the device using the Alif SETOOLS. This process only has to be done once.

Refer to the section [Usage](https://github.com/alifsemi/alif_ensemble-cmsis-dfp/blob/main/docs/Overview.md#usage)
in the overview page of the Alif Semiconductor Ensemble DFP/BSP for information on how
to setup these tools.

In VS Code use the menu command **Terminal - Run Tasks** and execute:

- "Alif: Install M55_HE and M55_HP debug stubs (dual core configuration)"

# Steps to build and debug the Zephyr example
1. Open the folder *./Zephyr-Alif* in VS code.
2. Initialize and update the Zephyr workspace, if none exists:
   - Use the menu command **Terminal - Run Tasks** and execute:
      - "Setup Zephyr-Alif workspace"
   - Now the required Zephyr and Alif modules can be found under the *./Zephyr-Alif/Zephyr-Workspace* folder.
3. To use the west tools for projects located in different folders:
   - Set ```ZEPHYR_BASE``` to ```C:/.../Zephyr-Workspace/zephyr```, and restart the VS Code.
5. In CMSIS View, press the `Select Active Solution from workspace` button to select the example you want.
6. Press the **"Build solution"** button to build the example:
   - The entire executed ```west build``` command can be found in the csolution.yml file under the **"executes:"** node.
   - The **"image:"** node specifies the path to the Zephyr image, which is then passed to the launch.json file for the debugger.
7. Start the CMSIS Debugger for dual-core debugging:
   - Start the **M55_HE CMSIS_DAP@pyOCD (launch)** debug session first, followed by the **M55_HP CMSIS_DAP@pyOCD (attach)** debug session.
   - After starting the second debug session, the program will halt at `cpu_idle.S`. This occurs because the second core still remains in its idle loop until a valid entry point is reached. To resolve this, add the following command under the **M55_HP CMSIS_DAP@pyOCD (attach)** section in the launch.json file:
   ```bash
   "initCommands": [
      "monitor reset halt",
      "tbreak main"
   ]
   ```
   - And remember to set `updateConfiguration:` to `manual` in order to prevent your settings from being overwritten.
