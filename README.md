# Zephyr-Alif (Work in Progress)

This repository contains an exemplary CMSIS solution file that can be used to build two Zephyr basic examples on
Alif development boards. It can be easily adapted to other boards or examples. It uses Zephyr's `west` build
system to create the executable file for an application and the
[Arm CMSIS Debugger](https://marketplace.visualstudio.com/items?itemName=Arm.vscode-cmsis-debugger) to flash download
and run the image on the target hardware.

## Quick start

- Clone this repository onto your machine.
- [Install Zephyr SDK-Alif](#zephyr-installation).
- Open this repository in VS Code. It should install required extensions automatically.
- In the CMSIS view, click on **...**, use **Open Solution in Workspace**, and choose your desired project.
- Set the environment variables for Zephyr workspace and Python virtual environoment as explained in the [Keil Studio documentation](https://mdk-packs.github.io/vscode-cmsis-solution-docs/zephyr.html#set-environment-variables).
- Make sure to fully restart the VS Code.
- Press the **Manage Solution Settings** button. In the dialog, select the target board and application.
- Press the **Build solution** button to build the example.
- Press the **Load & Debug application** button to start a debug session.
- [Work with the example](#work-with-the-example).

> [!NOTE]
> Check that the **Arm CMSIS Solution** extension is at least v1.64.0.

> [!WARNING]
> Do not install Python 3.14 or above as the Python package `windows-curses` is not yet available!

## Zephyr installation

This chapter contains installation instructions for [Linux and macOS](#linux-and-macos) and [Windows](#windows).

### Linux and macOS

- In your home directory, create a `sdk-alif` directory and change into it:

  ```sh
  mkdir sdk-alif
  cd sdk-alif
  ```

- Create a new virtual environment:

  ```sh
  python3 -m venv .venv
  ```

- Activate the virtual environment:

  ```sh
  source .venv/bin/activate
  ```

  Once activated, your shell will be prefixed with (.venv). The virtual environment can be deactivated at any time by
  running `deactivate`.

- Install west:

  ```sh
  pip install west
  ```

- Get the Zephyr source code using the Alif SDK:

  ```sh
  west init -m https://github.com/alifsemi/sdk-alif.git --mr v2.0.0
  west update
  ```

- Install Python dependencies using west packages:

  ```sh
  west packages pip --install
  ```

### Windows

- Open a `cmd.exe` terminal window as a regular user.

- Create a `sdk-alif` directory and change into it:

  ```sh
  mkdir sdk-alif
  cd sdk-alif
  ```

- Create a new virtual environment:

  ```sh
  python -m venv .venv
  ```

- Activate the virtual environment:

  ```sh
  .venv\Scripts\activate.bat
  ```

  Once activated your shell will be prefixed with (.venv). The virtual environment can be deactivated at any time by running deactivate.

- Install west:

  ```sh
  pip install west
  ```

- Get the Zephyr source code using the Alif SDK:

  ```sh
  west init -m https://github.com/alifsemi/sdk-alif.git --mr v2.0.0
  west update
  ```

- Install Python dependencies using west packages.

  ```sh
  west packages pip --install
  ```

## SETOOLS

Before flashing this example on the AppKit E7 board it is required to program the ATOC of the device using the Alif
SETOOLS. This process only has to be done once for each single- or dual-core project.

Refer to the section [Usage](https://github.com/alifsemi/alif_ensemble-cmsis-dfp/blob/main/docs/Overview.md#usage)
in the overview page of the Alif Semiconductor Ensemble DFP/BSP for information on how to setup these tools.

In VS Code use the menu command **"Terminal - Run Tasks"** and execute:

- "Alif: Install M55_HE and M55_HP debug stubs (dual core configuration)"

## Examples Description

| Example name                              | Description   |
|---                                        |---            |
| [IPM_ARM_MHUv2](./Examples/IPM_ARM_MHUv2/) | This example demonstrates the dual-core debugging and inter-core communication between two Cortex-M55 cores using Arm's Message Handling Unit v2 (MHUv2) . Also, it showcases the use of Zephyr's Inter-Processor Mailbox (IPM) API to exchange messages and trigger interrupts between cores. More details in [Alif-Zephyr-SDK/ipm_arm_mhuv2](https://github.com/alifsemi/sdk-alif/tree/v2.0.0/samples/drivers/ipm/ipm_arm_mhuv2). |
| [LPI2C](./Examples/LPI2C/) | LPI2C (Low Power Inter-Integrated Circuit) is a low-power version of the standard I2C bus controller. This example uses two threads to emulate an I2C master and slave communicating through the LPI2C driver. All data transfers occur internally via hardware loopback. More details in [Alif-Zephyr-SDK/lpi2c](https://github.com/alifsemi/sdk-alif/tree/v2.0.0/samples/drivers/lpi2c). |

## Work with the example

When working on dual-core projects,
- Start the **M55_HP CMSIS_DAP@pyOCD (launch)** debug session first, followed by **M55_HE CMSIS_DAP@pyOCD (attach)**.
- After starting the second debug session, the program will halt at `cpu_idle.S`. This occurs because the second core
  still remains in its idle loop until a valid entry point s reached. To resolve this, add the following command under
  the **M55_HE CMSIS_DAP@pyOCD (attach)** section in the launch.json file:

  ```bash
  "initCommands": [
     "monitor reset halt",
     "tbreak main"
  ]
  ```

- And remember to set `updateConfiguration:` to `manual` in order to prevent your settings from being overwritten.
