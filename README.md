# iMX 7ULP Exploration for Wearable Ultrasound Sensor

Using the NXP i.MX 7 ULP devkit, I evaluate the feasibility, performance and power consumption of a wearable ultrasound sensor with a power consumption under 1 Watt using the heterogeneous multi-core (Cortex-M4 and Cortex-A7) i.MX 7ULP chip.

## Installation and setup for Cortex-M4 application development

DISCLAIMER: the setup has been tested solely on Windows.

The vendor (NXP) does not provide a SDK compatible with MCUXpresso IDE. To debug/flash the i.MX7 Cortex-M4 processor without having to go through the tedious process of using u-boot to flash the image through QSPI, we prepared a tailored custom vscode configuration. Users don't need to copy the image on the SD Card to run their program but rather can use the J-LINK debugging probe.

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
- Build i.MX7 Cortex-M4 FW (output in src/armgcc/debug)
- Flash STM32 FW (requires a build)

### Pin multiplexing for FlexIO0 demo (SPI master -> LPSPI0 slave example, using eDMA)

Beware the uncorrect readme that has been provided in the src folder by NXP. FLEXIO0_1 and FLEXIO0_0 connection pads have been inverted. Please find the correct mapping below.

|  FlexIO0  | Alternate function |                Net               |  LPSPI0  |
|:---------:|:------------------:|:--------------------------------:|:--------:|
| FLEXIO0_0 |   PTA16 I2C0_SCL   |    J8-6 / R189 short-c / TP27    |  SPI_CLK |
| FLEXIO0_1 |   PTA17 I2C0_SDA   |    J8-5 / R188 short-c / TP25    | SPI_MOSI |
| FLEXIO0_4 |  PTA20 BATT_ADC_IN |  J8-4 / R190 short-c / R22 pad 1 | SPI_MISO |
| FLEXIO0_7 |   PTA23 ADC_INT#   | J8-3 / R191 short-c / R101 pad 1 |  SPI_CS  |

### Execute code on i.MX7 Cortex-M4 using JLINK

I have found that to succesfully attach the debugger on i.MX7ULP-EVK every time, the DIP switches configuration must be as follow:

| DIP PIN | [0] | [1] | [2] | [3] |
|:-------:|:---:|:---:|:---:|:---:|
|  State  |  0  |  1  |  0  |  0  |

Connect the JTAG 20 pin connector on the EVK to JLINK. Power the JLink adapter. Use the build i.MX7 task to compile the software, and then press F5 to run i.MX7ULP Cortex-M4 Debug configuration. The PC halts at the main function.

### Execute code on STM32-Nucleo-MB1136 using STLINK (on-board debugger)

Connect the STM32 board through USB. Build the STM32 FW using the task provided in the VS Workspace. Press F5 and run STM32 Cortex-M4 Debug configuration.

### Application developmnent on Linux (Under construction..)

Instructions to prepare your development computer (host) for Cortex-A7 are put together in notes.md.

#### Modify the app' and recompile

If you thoroughly followed the steps in notes.md, open a terminal in the base directory.

Clone this github repository in the base directory. The steps are detailed and extracted from [this google document](https://drive.google.com/file/d/0B3KGzY5fW7laTDVxUXo3UDRvd2s/view?resourcekey=0-vkLVQQMlhHdH61NGJzmLRg).

```bash
devtool add fpga2wifi <path-to-basedirectory>/<this-repo>/imx7a7
```

In `build/conf/local.conf`, append this line to add the recipe/layer to the build configuration:

```bash
IMAGE_INSTALL_append= " fpga2wifi"
```

Add your modifications in the c files... and then run `bitbake fpga2wifi` to recompile just this layer or rather the whole image with: 

```bash
bitbake fsl-image-machine-test
```

Then, reflash the SD card (see additonal steps in notes.md) with the new image. Run the application by typing the name in the linux console.
(Use PuTTY, Tera Term, minicom or picocom to start a UART communication (baudrate 115200) through USB)

#### Improvements

```txt
imx7-ulp-fpga2wifi-yoctobase (manifest for the repo command)
meta-fpga2wifi (application code for Cortex A7)
imx7-ulp-cortexm4 (application code for M4 domain)
```

A better organization would be to have a separate repository for the manifest (containing the base image from i.MX + the custom A7 application), a repository for the application code only, and this repository for real-time domain (so we don't clone the whole project and 300 Mb image on the development host...).

## Documentation

A compilation of useful readings, board design files, and user guides is stored into the `documentation` folder.

[NXP Quick start guide (online)](https://www.nxp.com/document/guide/get-started-with-the-mcimx7ulp-evk:GS-MCIMX7ULP-EVK)

Yocto Project:

- [Yocto SDK Manual (application development workflow)](https://docs.yoctoproject.org/sdk-manual/intro.html#introduction)
- [Yocto Workflow](https://drive.google.com/file/d/0B3KGzY5fW7laTDVxUXo3UDRvd2s/view?resourcekey=0-vkLVQQMlhHdH61NGJzmLRg)

Bootlin company:

- [Entry point for their training material](https://bootlin.com/docs/)
- [Yocto training (course)](https://bootlin.com/doc/training/yocto/yocto-slides.pdf)
- [Yocto training (lab)](https://bootlin.com/doc/training/yocto/yocto-labs.pdf)

Toradex knowledge-base can be helpful to cross-check your steps.

- [Setup the environment for Embedded Linux application development](https://developer.toradex.com/knowledge-base/how-to-setup-environment-for-embedded-linux-application-development#Network_Connectivity)
- [Hello World application on Embedded Linux](https://developer.toradex.com/knowledge-base/hello-world-application-on-embedded-linux)
- [Linux SDKs](https://developer.toradex.com/knowledge-base/linux-sdks)

## Contact

Reach out to me (Guillaume Thivolet) at <guillaume@glabs.ch>.
