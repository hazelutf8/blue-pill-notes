# STM32 Blue Pill Notes  

## Environment Setup  

### Arduino IDE - Windows

These instructions are targeted at Windows 7+, and assume nothing is yet installed (fresh install).  
The majority of the setup is to accommodate the tools which interact with the non-native HW platform.

------

#### Install Arudino IDE  

Go to the Arduino website and [download](https://www.arduino.cc/en/Main/Software) the IDE installer.
Install the Arduino IDE.

Add the ARM compiler to the IDE through the menu:
- **File** --> **Preferences** --> **Additional Boards Manager URLs**
- Insert/Add `http://dan.drown.org/stm32duino/package_STM32duino_index.json`
- **Tools** --> **Board** --> **Boards Manager**
- Search for "STM32F1xx" to find "STM32F1xx/GD32F1xx boards" by "stm32duino", and click install.


#### (Optional) Install Git for Windows  

Install `git` for interaction with github and version control project code.  
- Git for windows [homepage](https://gitforwindows.org/)


#### Install ST-Link Software

- Unfortunately STMicroelectronics requires a free account to be created for [st.com](https://www.st.com/) downloads.  

- Download and install the ST-Link v2 dongle driver STLinkWinUSB [stsw-link0004](https://www.st.com/en/development-tools/stsw-link009.html).  
  This allowes the dongle to be a recognized device in windows, but doesn't install any debugging/flashing software.  

- Download and install the ST-LINK Utility [stsw-link009](https://www.st.com/en/development-tools/stsw-link004.html).  
  The Arudino IDE uses the `ST-LINK_CLI` utility to flash over SWD/JTAG without a bootloader.


#### (Optional) Install Debugging Server

Get the `openocd` tool from the `xpm` package manager (and not the deprecated github releases page).
- Download Node.js [homepage](https://nodejs.org/en/download/)
- Install `openocd` for windows from the latest maintainer, the `xpm` repository.
  ```batch
  npm install --global xpm
  xpm install --global @gnu-mcu-eclipse/arm-none-eabi-gcc
  ```
- Add it to the global PATH variable.  
  Example: `%APPDATA%\xPacks\@xpack-dev-tools\openocd\0.10.0-13.1\.content\bin`  
  - Add it to your global PATH variable.  
    - Open the "Environment Variables" menu by typing:  
      `<Win> + <r>`  
      `rundll32.exe sysdm.cpl,EditEnvironmentVariables`  
    - Edit/insert the PATH variable with the found path.
- Verify  
  Open a new `cmd.exe` window and execute the command:
  ```batch
  openocd --version
  ```

#### (Optional) Install GDB Debugger  

Add the Arduino IDE version of GDB to the environment PATH.
- Locate where `arm-none-eabi-gdb` is stored for the Arduino IDE.  
  Example: `%LOCALAPPDATA%\Arduino15\packages\arduino\tools\arm-none-eabi-gcc\4.8.3-2014q1\bin`  
- Add it to the global PATH variable.  
  - Open the "Environment Variables" menu by typing:  
    `<Win> + <r>`  
    `rundll32.exe sysdm.cpl,EditEnvironmentVariables`  
  - Edit/insert the PATH variable with the found path.
- Verify  
  Open a new `cmd.exe` window and execute the command:
  ```batch
  arm-none-eabi-gdb --version
  ```

#### Configure Arudino IDE Board

Select the correct board/flasher combination.
- **Tools** --> **Board** --> **Generic STM32F103C series**
- **Tools** --> **Variant** --> Choose the correct flash size based on your board (usually 128Kb).
- **Tools** --> **Upload Method** --> **STLink**
- If you don't see "STLink" as an upload method reboot the computer or re-install the ST-Link software, or else you can't flash the board over SWD/JTAG.
- **Tools** --> **CPU Speed** --> **72MHz (Normal)**
- **Tools** --> **Optimize** --> **Smallest (Normal)**
- The optimization options can be tweaked as required.

#### (Optional) Erase Device Flash

Using the ST-Link Utility software you can erase the entire flash.
- Launch the "STM32 ST-Link Utility"
- Connect to the Blue-Pill **Target** --> **Connect**
- Observe the "Device Memory" flash data is shown.
- Click the eraser icon, or **Target** --> **Erase Chip**
- Observe all "Device Memory" flash data is all "FFFFFFFF" values, and any previous blink program on the device does not run.

#### Verify Arudino IDE Works

Open the Example blink program in the Arduino IDE.
- **File** --> **Examples** --> **01.Basics** --> **Blink**
- Ensure the ST-Link v2 dongle is connected to both the computer and the Blue-Pill board.  
  Note, there are two variants of the 10-pin ST-Link v2 dongle (with "M" or "ST" logos), which have the same pins but slightly different pinout locations.

| Blue-Pill SWD/JTAG | ST-Link v2 Dongle | 
| ------------- |-------------|
| 3V3 | 3.3V |
| SWIO | SWDIO |
| SWCLK | SWCLK |
| GND | GND |

- Click the "Upload" button, which will compile, upload, and reset the device.
- If the IDE says it is successful, change the delays and upload again just to double check the Blink pattern changes.

#### (Optional) Verify Debugger Works  

Open a command line terminal to launch the debugger server.
- Launch the OpenOCD server, which attaches to the ST-Link v2 dongle. 
  ```batch
  openocd -f interface\stlink-v2.cfg -f target\stm32f1x.cfg
  ```  
Open a new command line terminal, to launch the debugger.
- Find where the Arduino IDE created the output binary, and run it with the GDB debugger.
  Example: `%TEMP%\arduino_build_189674\blink_test.ino.elf`  
  ```bash
  echo Determine temp build location
  cd %TEMP%\arduino_build_<version>
  
  echo Show the built binaries:
  dir *.elf
  
  echo Launch GDB on your binary:
  arm-none-eabi-gdb <your_custom_binary.elf>
  ```
- Attach the debugger to the OpenOCD server and run the program.
  ```gdb
  # Connect to the OpenOCD Server
  target remote :3333

  # (Optional) print demangled symbols
  set print asm-demangle on
  
  # (Optional) Load the binary to the Blue-Pill board
  load
  
  # (Optional) Insert common breakpoints on function names
  break main
  break loop
  
  # Run the program on the Blue-Pill
  continue
  
  # Continue after hitting a breakpoint (you can abbrivate commands)
  c
  ```
- Common GDB commands [cheat-sheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf).
- (Optional) GDB commands can be saved to a file (ex: `openocd_gdb_connect.gdb`) and run at startup.
  ```batch
  arm-none-eabi-gdb -x openocd_gdb_connect.gdb <your_custom_binary.elf>
  ```

------

### Tested Versions
| Name | Version | 
| ------------- | ------------- |
| `Arduino IDE` | `1.8.12` |
| `arm-none-eabi-gdb --version` | `GNU gdb (GNU Tools for ARM Embedded Processors) 7.6.0.20140228-cvs` |
| `Blink.ino` | `modified 8 Sep 2016` |
| `Blue-Pill` | `STM32F103C8T6` |
| `git --version` | `git version 2.23.0.windows.1` |
| `npm --version` | `6.11.3` |
| `openocd --version` | `xPack OpenOCD, 64-bit Open On-Chip Debugger 0.10.0+dev (2019-07-17-11:28)` |
| `"STM32F1xx/GD32F1xx boards" by stm32duino` | `version 2020.2.19` |
| `ST-LINK V2 dongle` | `Logo has "M" icon` |
| `Windows 10 Home` | `Version 1903 OS build 18362.657` |
| `xpm --version` | `0.5.0` |

------
