# iMX 7ULP Exploration for Wearable Ultrasound Sensor

This project is conducted for the SoCDaML course exmanitation as a miniproject.

Using the NXP i.MX 7 ULP devkit, we evaluate the feasibility, performance and power consumption of a wearable ultrasound sensor with a power consumption under 1 Watt.

## Project planification

[Gantt chart](https://view.monday.com/1453663162-e9185251b7c593d24864bfbab5e83f3e?r=use1) (outdated)

## Installation and setup for Cortex-M4 application development

DISCLAIMER: the setup has been tested on Windows.

The vendor (NXP) does not provide a SDK compatible with MCUXpresso IDE. To debug/flash the IMX7 Cortex-M4 processor without having to go through the tedious process of using u-boot to flash the image through QSPI, we prepared a tailored custom toolchain. Users don't need to copy the image on the SD Card to run their program but rather can use J-LINK.

Firstly, follow the [procedure](https://github.com/NXPmicro/mcux-sdk/blob/main/docs/run_a_project_using_armgcc.md) given by NXP to install armgcc.

Then, the user should install [Visual Studio Code](https://code.visualstudio.com/) (latest version) and the Cortex Debug extension on VSCode. ![Cortex Debug](readme-images/cortex.png)

Open the preferences in VSCode (or use <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>P</kbd> and type `pref`), and append the following code. Modify accordingly to match your local installation.

```json
    "cortex-debug.JLinkGDBServerPath": "C:\\Program Files (x86)\\SEGGER\\JLink\\JLinkGDBServerCL.exe",
    "cortex-debug.armToolchainPath": "C:\\Program Files (x86)\\GNU Arm Embedded Toolchain\\10 2020-q4-major\\bin"
```

### VSCode workspace

|         File         |              Description             |
|:--------------------:|:------------------------------------:|
|  .vscode/tasks.json  |      VSCode tasks configuration      |
| .vsccode/launch.json | Cortex-Debug extension configuration |

Tasks and launch configurations have already been written and don't require modifications.

The keyboard shortcut to run tasks is <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>B</kbd>. Three options are available:

- Build STM32 FW (output in stm32nucleo/spi/build)
- Build IMX7 Cortex-M4 FW (output in src/armgcc/debug)
- Flash STM32 FW (requires a build)

### Execute code on IMX7 Cortex-M4 using JLINK

To succesfully attach the debugger on IMX7ULP-EVK, the DIP switches configuration must be as follow:

| DIP PIN | [0] | [1] | [2] | [3] |
|:-------:|:---:|:---:|:---:|:---:|
|  State  |  0  |  1  |  0  |  0  |

Connect the JTAG 20 pin connector on the EVK to JLINK. Power the JLink adapter. Use the build IMX7 task to compile the software, and then press F5 to run IMX7ULP Cortex-M4 Debug configuration.

### Execute code on STM32-Nucleo-MB1136 using STLINK (on-board debugger)

Connect the STM32 board through USB. Build the STM32 FW using the task provided in the VS Workspace. Press F5 and run STM32 Cortex-M4 Debug configuration. 

### Pin multiplexing

Beware the uncorrect readme that has been provided in the src folder by NXP. FLEXIO0_1 and FLEXIO0_0 connection pads have been inverted. Please find the correct mapping below.

|  FlexIO0  | Alternate function |                Net               |  LPSPI0  |
|:---------:|:------------------:|:--------------------------------:|:--------:|
| FLEXIO0_0 |   PTA16 I2C0_SCL   |    J8-6 / R189 short-c / TP27    |  SPI_CLK |
| FLEXIO0_1 |   PTA17 I2C0_SDA   |    J8-5 / R188 short-c / TP25    | SPI_MOSI |
| FLEXIO0_4 |  PTA20 BATT_ADC_IN |  J8-4 / R190 short-c / R22 pad 1 | SPI_MISO |
| FLEXIO0_7 |   PTA23 ADC_INT#   | J8-3 / R191 short-c / R101 pad 1 |  SPI_CS  |

## Documentation

A compilation of useful readings, board design files, and user guides is stored into the `documentation` folder.

[Gitlab (online)](https://iis-git.ee.ethz.ch/gthivolet/imx7ulp-exploration)

[NXP Quick start guide (online)](https://www.nxp.com/document/guide/get-started-with-the-mcimx7ulp-evk:GS-MCIMX7ULP-EVK)

Bootlin company:

- [Entry point for their training material](https://bootlin.com/docs/)
- [Yocto training (course)](https://bootlin.com/doc/training/yocto/yocto-slides.pdf)
- [Yocto training (lab)](https://bootlin.com/doc/training/yocto/yocto-labs.pdf)

Toradex knowledge-base can be helpful for cross-checking your steps.

- [Setup the environment for Embedded Linux application development](https://developer.toradex.com/knowledge-base/how-to-setup-environment-for-embedded-linux-application-development#Network_Connectivity)
- [Hello World application on Embedded Linux](https://developer.toradex.com/knowledge-base/hello-world-application-on-embedded-linux)
- [Linux SDKs](https://developer.toradex.com/knowledge-base/linux-sdks)

## Contact

Reach out to me (Guillaume Thivolet) at <guillaume@glabs.ch>.
