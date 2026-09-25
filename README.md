# Zephyr-Alif (Work in Progress)

This repository contains an example CMSIS solution that builds two basic Zephyr examples for Alif development boards.
It can be easily adapted to other boards or examples. It uses Zephyr's `west` build system to create the application
image and the [Arm CMSIS Debugger](https://marketplace.visualstudio.com/items?itemName=Arm.vscode-cmsis-debugger) to
download the image to flash memory and run it on the target hardware.

## Quick start

- Install the [Arm Keil Studio Pack for VS Code](https://marketplace.visualstudio.com/items?itemName=Arm.keil-studio-pack),
  then clone this repository and open its folder in VS Code.
- [Install Alif Zephyr SDK 2.3](#zephyr-installation) and [configure its environment variables](#configure-vs-code).
- Restart VS Code, then select **CMSIS > ... > Open Solution in Workspace** and open an example's
  `*.csolution.yml` file.
- Use **Manage Solution Settings** to select the board and application, then **Build solution**.
- Connect the board and select **Load & Debug application**. See [Work with the example](#work-with-the-example) for
  dual-core projects.

> [!NOTE]
> Ensure that the **Arm CMSIS Solution** extension is version 1.72.0 or later.

## Zephyr installation

The following instructions apply to Linux, macOS, and Windows. Install a supported version of Python 3 and Git before
continuing.

> [!WARNING]
> On Windows, use Python 3.13 or earlier because the `windows-curses` package is not yet available for Python 3.14.

### Create the workspace

- Open a terminal as a regular user. On Windows, use `cmd.exe` for the commands below.

- In any suitable working directory, create an `sdk-alif` directory and change into it:

  ```console
  mkdir sdk-alif
  cd sdk-alif
  ```

- Create a virtual environment. Use the command for your operating system:

  Linux and macOS:

  ```sh
  python3 -m venv .venv
  ```

  Windows:

  ```bat
  python -m venv .venv
  ```

- Activate the virtual environment.

  Linux and macOS:

  ```sh
  source .venv/bin/activate
  ```

  Windows (`cmd.exe`):

  ```bat
  .venv\Scripts\activate.bat
  ```

  Once activated, the shell prompt is prefixed with `(.venv)`. Activate the environment again whenever you open a
  new terminal. Run `deactivate` to leave it.

- Install west. After activation, `python` refers to the virtual environment on every supported operating system:

  ```console
  python -m pip install west
  ```

- Get the Zephyr source code using the Alif SDK:

  ```console
  west init -m https://github.com/alifsemi/sdk-alif.git --mr v2.3.0
  west update
  ```

- Install the Python dependencies required by Zephyr:

  ```console
  python -m pip install -r zephyr/scripts/requirements.txt
  ```

### Configure VS Code

The CMSIS Solution extension needs the Zephyr workspace and virtual environment paths when it runs `west`.

1. In VS Code, open **Settings** and search for **Cmsis-Csolution: Environment Variables**.
2. Select the **User** or **Workspace** setting and choose **Add Item** for each variable below. Replace the example
   prefix with the absolute path to your `sdk-alif` directory.

   | Variable | Linux and macOS | Windows |
   |---|---|---|
   | `ZEPHYR_BASE` | `/work/sdk-alif/zephyr` | `C:\work\sdk-alif\zephyr` |
   | `PATH` | `/work/sdk-alif/.venv/bin` | `C:\work\sdk-alif\.venv\Scripts` |
   | `VIRTUAL_ENV` | `/work/sdk-alif/.venv` | `C:\work\sdk-alif\.venv` |

3. Fully restart VS Code so that the extension uses the new environment.

For more information, see [Work with Zephyr applications](https://mdk-packs.github.io/vscode-cmsis-solution-docs/zephyr.html#set-environment-variables).

## SETOOLS

Before flashing an example to the AppKit E7 board, program the device's ATOC using Alif SETOOLS. This process only has
to be performed once for each single- or dual-core project.

Refer to the section [Usage](https://github.com/alifsemi/alif_ensemble-cmsis-dfp/blob/main/docs/Overview.md#usage)
on the Alif Semiconductor Ensemble DFP/BSP overview page for information about how to set up these tools.

In VS Code, select **Terminal > Run Task** and run:

- **Alif: Install M55_HE and M55_HP debug stubs (dual core configuration)**

## Example descriptions

| Example name                              | Description   |
|---                                        |---            |
| [IPM_ARM_MHUv2](./Examples/IPM_ARM_MHUv2/) | This example demonstrates dual-core debugging and inter-core communication between two Cortex-M55 cores using Arm's Message Handling Unit v2 (MHUv2). It uses Zephyr's Inter-Processor Mailbox (IPM) API to exchange messages and trigger interrupts between the cores. More details in [Alif-Zephyr-SDK/ipm_arm_mhuv2](https://github.com/alifsemi/sdk-alif/tree/v2.3.0/samples/drivers/ipm/ipm_arm_mhuv2). |
| [LPI2C](./Examples/LPI2C/) | LPI2C (Low Power Inter-Integrated Circuit) is a low-power version of the standard I2C bus controller. This example uses two threads to emulate an I2C master and slave communicating through the LPI2C driver. All data transfers occur internally via hardware loopback. More details in [Alif-Zephyr-SDK/lpi2c](https://github.com/alifsemi/sdk-alif/tree/v2.3.0/samples/drivers/lpi2c). |

## Work with the example

When working on a dual-core project:

- Start the **M55_HP CMSIS_DAP@pyOCD (launch)** debug session first, followed by **M55_HE CMSIS_DAP@pyOCD (attach)**.
- After starting the second debug session, the program will halt at `cpu_idle.S`. This occurs because the second core
  remains in its idle loop until it receives a valid entry point. To resolve this, add the following commands to the
  **M55_HE CMSIS_DAP@pyOCD (attach)** section in `launch.json`:

  ```bash
  "initCommands": [
     "monitor reset halt",
     "tbreak main"
  ]
  ```

- Set `updateConfiguration` to `manual` to prevent your settings from being overwritten.
