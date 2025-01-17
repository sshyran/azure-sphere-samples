---
page_type: sample
languages:
- c
products:
- azure
- azure-sphere
name: Azure Sphere – External MCU update
urlFragment: ExternalMcuUpdate
extendedZipContent:
- path: HardwareDefinitions
  target: AzureSphere_HighLevelApp/HardwareDefinitions
- path: .clang-format
  target: .clang-format
- path: BUILD_INSTRUCTIONS.md
  target: BUILD_INSTRUCTIONS.md
- path: Samples/SECURITY.md
  target: SECURITY.md
- path: Samples/troubleshooting.md
  target: troubleshooting.md
description: "Demonstrates how you might use an Azure Sphere device to deploy an update to an external MCU device."
---

# Sample: External MCU update

Your product may incorporate other MCUs with your Azure Sphere device, and [those other MCUs may require updates](https://docs.microsoft.com/azure-sphere/deployment/external-mcu-update). Assuming the other MCUs permit updates to be loaded over the connection you establish with the Azure Sphere device, for example over UART, you can use the Azure Sphere device to securely deliver those updates.

This reference solution demonstrates how you might use an Azure Sphere device to deploy an update to an external MCU device. This solution contains an Azure Sphere app that deploys firmware to the Nordic nRF52 Development Kit over UART. This app can itself be updated remotely via [over-the-air updates](https://docs.microsoft.com/azure-sphere/deployment/deployment-overview), ensuring that the software versions of this app and the MCU firmware are always in sync.

## Preparation

This reference solution requires the following:

- Azure Sphere SDK version 21.01 or above. At the command prompt, run **azsphere show-version** to check. Download and install the latest SDK as needed. The [set-up procedures](https://docs.microsoft.com/azure-sphere/install/overview) describe how to choose and install an SDK.
- Azure Sphere MT3620 board
- Nordic nRF52 BLE development board
- Jumper wires to connect the boards
- Two free USB ports on your computer

**Note:** By default, this sample targets [MT3620 reference development board (RDB)](https://docs.microsoft.com/azure-sphere/hardware/mt3620-reference-board-design) hardware, such as the MT3620 development kit from Seeed Studios. To build the sample for different Azure Sphere hardware, change the Target Hardware Definition Directory in the CMakeLists.txt file. For detailed instructions, see the [README file in the HardwareDefinitions folder](../../HardwareDefinitions/README.md). You might also need to wire the device differently.

## Connect Azure Sphere MT3620 to the Nordic nRF52

Make the following connections between the nRF52 and MT3620 dev boards using the jumper wires:

- nRF52 RX: P0.11 to MT3620 UART0 TX: Header 2 (lower left) Pin 3
- nRF52 TX: P0.12 to MT3620 UART0 RX: Header 2 (lower left) Pin 1
- nRF52 CTS: P0.23 to MT3620 UART0 RTS: Header 2 (lower left) Pin 7
- nRF52 RTS: P0.22 to MT3620 UART0 CTS: Header 2 (lower left) Pin 5
- nRF52 Reset: P0.21 to MT3620 GPIO5: Header 2 (lower left) Pin 4
- nRF52 DFU: P0.16 to MT3620 GPIO44: Header 2 (lower left) Pin 14
- nRF52 Ground: GND to MT3620 GND: Header 2 (lower left) Pin 2

Refer to the following graphic for details.

![Connection diagram for nRF52 and MT3620](./media/nRF52_MT3620_connection.png)

## Install bootloader on the nRF52

1. Clone the [Azure Sphere samples](https://github.com/Azure/azure-sphere-samples) repository and find the *ExternalMcuUpdate* sample in the *ExternalMcuUpdate* folder or download the zip file from the [Microsoft samples browser](https://docs.microsoft.com/samples/azure/azure-sphere-samples/externalmcuupdate/).
1. Connect the nRF52 developer board to your computer using USB. Once connected, the nRF52 displays a JLINK removable drive in Windows.
1. Find softdevice_Bootloader.hex in the ExternalMcuUpdateNrf52\Binaries folder, and copy it to the JLINK drive. The nRF52 restarts automatically and runs the bootloader.
1. Observe that LED1 and LED3 are lit on the nRF52 development board, which indicates that the bootloader has started successfully. 

## Build and run the Azure Sphere app that updates the firmware on the nRF52

Follow the instructions in [Build a sample application](../../BUILD_INSTRUCTIONS.md) to build and run the Azure Sphere app in Visual Studio or Visual Studio Code.

### Observe the app while it updates the firmware on the nRF52

1. As the app runs, observe the Output window for activity messages. You should see the sample firmware install on the nRF52.
1. Observe that LED2 and LED4 are blinking on the nRF52 development board, which indicates the new firmware is running.
1. Press button A to restart the update process. In the Output window, observe that the app determines the nRF52 firmware is already up to date, and does not reinstall it.

## Edit the Azure Sphere app to deploy different firmware to the nRF52

The nRF52 firmware files are included as resources within the Azure Sphere app. The app can easily be rebuilt to include different firmware. For example, to run the BlinkyV2 nRF52 app you replace the BlinkyV1.bin and BlinkyV1.dat files with the corresponding BlinkyV2 files.

1. In the [CMakeLists.txt](./AzureSphere_HighLevelApp/CMakeLists.txt) file for this sample edit the azsphere_target_add_image_package command to replace BlinkyV1.bin and BlinkyV1.dat with BlinkyV2.bin and BlinkyV2.dat:

        azsphere_target_add_image_package(
        ${PROJECT_NAME}
         RESOURCE_FILES
        "ExternalNRF52Firmware/blinkyV2.bin"
        "ExternalNRF52Firmware/blinkyV2.dat"
        "ExternalNRF52Firmware/s132_nrf52_6.1.0_softdevice.bin"
        "ExternalNRF52Firmware/s132_nrf52_6.1.0_softdevice.dat")    

1. Update the filename constants in main.c to point at BlinkyV2 instead of BlinkyV1.
1. Update the accompanying version constant to '2' instead of '1'.
1. Ensure the "SoftDevice" BLE stack firmware files (s132_nrf52_6.1.0_softdevice.bin and s132_nrf52_6.1.0_softdevice.dat) are still included as resources. Do not edit the constants that relate to these files.
1. Rebuild and run the Azure Sphere app.
1. Use the Output window to observe as the BlinkyV2 firmware is installed and run on the nRF52.
1. Observe that LED3 and LED4 are now blinking on the nRF52 development board, which indicates that the BlinkyV2 firmware is running.
1. Notice that the SoftDevice BLE stack was not updated because it is already at the correct version.

## Build and deploy your own app firmware for the nRF52

You can adapt this solution to include your own nRF52 app.

### Create a new 'BlinkyV3' nRF52 app

1. Download and install [SEGGER Embedded Studio](https://www.segger.com/downloads/embedded-studio). [Download the 32-bit version](https://www.segger.com/downloads/embedded-studio/EmbeddedStudio_ARM_Win_x86), not the 64-bit version. Ensure that you are licensed to use it for your purposes. In Oct 2018, we were able to [obtain the license for free because we were developing for NRF52](https://www.segger.com/news/segger-embedded-studio-ide-now-free-for-nordic-sdk-users/).
1. Download and unzip the [Nordic NRF5 SDK V15.2](https://www.nordicsemi.com/eng/Products/Bluetooth-low-energy/nRF5-SDK#Downloads).
1. Edit Nordic's Blinky sample app so you can build it against your SDK:
    - Use a text editor to open <NORDIC_SDK_PATH>\examples\peripheral\blinky\pca10040\s132\ses\blinky_pca10040_s132.emProject.
    - Set the SDK_ROOT variable in this file to point to the root directory your Nordic SDK install. Specifically, replace the words "CHANGE_THIS_TO_YOUR_NORDIC_SDK_PATH" with the correct path, changing any backslashes ("\\") to forward slashes ("/") in the path. For example: macros="SDK_ROOT=C:/Users/ExampleUser/source/nRF5_SDK_15.2.0_9412b96;…"
1. Open this .emProject file in the Segger IDE.
1. Build it (**Build->Build Solution**) to generate a .hex file. It is placed in this location: <NORDIC_SDK_PATH>\examples\peripheral\blinky\pca10040\s132\ses\Output\Release\Exe\blinky_pca10040_s132.hex

### Obtain the BlinkyV3.bin and BlinkyV3.dat app firmware files

1. Install [Python](https://www.python.org/downloads/) 2.7.6 (32-bit) or later. **Note:** Python 3 won't work but it's fine to have this installed side-by-side.
1. Install the Nordic [nrfutil](http://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.tools%2Fdita%2Ftools%2Fnrfutil%2Fnrfutil_intro.html) CLI.
1. Use the **nrfutil** utility to transform the downloaded app .hex file into a firmware update package .zip file. Specify the app version in the command.
    - Open a command prompt and go to the directory that contains the .hex file generated above.
    - Run this command: 
       `nrfutil pkg generate --application blinky_pca10040_s132.hex --sd-req "0xA8","0xAF" --application-version 3 --hw-version 52 blinkyV3.zip` 
    - The version specified to pack the application ('3' in this case) must also be specified in the Azure Sphere app when you [deploy the new firmware](#deploy-the-new-firmware).
    - Note the warning about 'not providing a signature key'. This doesn't stop you from proceeding, but you should consider whether this is acceptable for your production scenario or whether, for example, your product is designed so that only the Azure Sphere chip has the ability to update the nRF52 firmware.
    - See the [nrfutil documentation](https://libraries.io/pypi/nrfutil) for more details about its command options.
1. Open the resulting .zip file and extract the .bin and metadata .dat back out from this .zip package.
1. Rename these files as desired—for example, BlinkyV3.bin/.dat.

### Deploy the new firmware

Add BlinkyV3.bin and BlinkyV3.dat as resources in the AzureSphere app by following the steps specified in [Edit the Azure Sphere app to deploy different firmware to the nRF52](#edit-the-azure-sphere-app-to-deploy-different-firmware-to-the-nrf52). Remember to update the filenames and the version to '3' in main.c.

## Combine this solution with the solution for BLE-based Wi-Fi setup

You can combine this solution for external MCU update with the solution for [BLE-based Wi-Fi setup](https://github.com/Azure/azure-sphere-samples/tree/master/Samples/WifiSetupAndDeviceControlViaBle). Doing so allows you to remotely update that solution's nRF52 application.

### Create a single Azure Sphere application

1. Remove the code that uses button A to trigger a firmware update and instead trigger it only when the combined app starts. In particular, remove the DfuTimerEventHandler, dfuButtonTimerEvent, gpioButtonFd, buttonPressCheckPeriod and dfuButtonTimerFd.
1. Combine the initialization and close functions. In particular, the UART, Epoll, and Reset file descriptors are opened and closed by both applications. Make sure you maintain only a single copy of each.
1. After the MCU update is complete, make sure to remove any of its UART event handlers from the epoll event loop. This will allow the Wi-Fi setup code to register its own UART event handlers when needed. Failure to do so will result in such registrations returning -1 with errno set to EEXISTS.
1. Combine the app manifest. In particular, add the required GPIOs for both applications and set the WifiConfig capability to true.

### Obtain the nRF52 firmware files

1. Rebuild the nRF52 firmware for the [BLE-based Wi-Fi app](https://github.com/Azure/azure-sphere-samples/tree/master/Samples/WifiSetupAndDeviceControlViaBle#build-your-own-solution). Select **Build->Build Solution** or press F7 to generate a .hex file. The hex file is placed in this location: 
 
   <PATH_TO_DOWNLOADED_SAMPLES>\WifiSetupAndDeviceControlViaBle\Nrf52App\pca10040\s132\ses\Output\Release\Exe\ble_app_uart_pca10040_s132.hex

1. Follow the steps specified in [Obtain the BlinkyV3.bin and BlinkyV3.dat app firmware files](https://github.com/Azure/azure-sphere-samples/tree/master/Samples/ExternalMcuUpdate#obtain-the-blinkyv3bin-and-blinkyv3dat-app-firmware-files) to transform this .hex file into .dat and .bin files.
1. Rename these fields as desired—for example, WifiSetupAndDeviceControlViaBle.bin/.dat

### Edit the Azure Sphere application to deploy the new nRF52 firmware

Add WifiSetupAndDeviceControlViaBle.bin and WifiSetupAndDeviceControlViaBle.dat as resources in the ExternalMcuUpdateNrf52 app by following the steps specified in [Edit the Azure Sphere app to deploy different firmware to the nRF52](#edit-the-azure-sphere-app-to-deploy-different-firmware-to-the-nrf52). Remember to update the filenames and the version in main.c. For example, if you've previously deployed version '3' of the firmware (even if it was just test firmware such as 'Blinky') then the version should now be '4'.

## Build your own bootloader

This sample includes a modified version of the example bootloader (secure_bootloader\pca10040_uart_debug) in the nRF5 SDK. It has been modified to:

- Use a custom board configuration where the UART pins are remapped and have pull-up resistors enabled on the input pins.
- Accept signed or unsigned applications—consider whether this is acceptable for your production scenario.
- Accept signed or unsigned bootloaders—consider whether this is acceptable for your production scenario.
- Accept firmware upgrades or downgrades.
- Enable Device Firmware Update (DFU) mode via pin input, as well as by pressing the Reset button on the nRF52 board.

To further edit and deploy this bootloader:

1. Edit the bootloader sample so you can build it against your SDK:
    - Use a text editor to open ExternalMcuUpdateNrf52\Nrf52Bootloader\pca10040\s132\ses\bootloader_uart_mbr_pca10040_debug.emProject.
    - Set the SDK_ROOT variable in this file to point to the root directory your Nordic SDK install. Specifically, replace the words "CHANGE_THIS_TO_YOUR_NORDIC_SDK_PATH" with the correct path, changing the backslashes ("\\") to forward slashes ("/") in the path. For example: macros="SDK_ROOT=C:/Users/ExampleUser/source/nRF5_SDK_15.2.0_9412b96;…"
1. Open this .emProject file in the Segger IDE.
1. Build it (**Build->Build Solution**) to generate a .hex file. It is placed in this location: ExternalMcuUpdateNrf52\Nrf52Bootloader\pca10040\s132\ses\Output\Release\Exe\bootloader_uart_mbr_pca10040_debug.hex
1. Copy this file to the JLINK drive presented by the nRF52. The nRF52 restart automatically and runs the bootloader.

## License
For license details, see LICENSE.txt in each directory.

## Code of Conduct
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
