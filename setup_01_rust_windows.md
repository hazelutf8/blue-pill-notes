# STM32 Blue Pill Notes  

## Environment Setup

### Rust - Windows

These instructions are targeted at Windows 7+, and assume nothing is yet installed (fresh install).
The majority of the setup is to accommodate the tools which interact with the non-native HW platform.

------

#### Install Git for Windows

Needed for downloading example code from github or versioning project code.
- Git for Windows [homepage](https://gitforWindows.org/)


#### Install ST-Link Software

- Unfortunately STMicroelectronics requires a free account to be created for [st.com](https://www.st.com/) downloads.  

- Download and install the ST-Link v2 dongle driver STLinkWinUSB [stsw-link0004](https://www.st.com/en/development-tools/stsw-link009.html).  
  This allowes the dongle to be a recognized device in Windows, but doesn't install any debugging/flashing software.  

- (Optional) Download and install the ST-LINK Utility [stsw-link009](https://www.st.com/en/development-tools/stsw-link004.html).  
  The GUI can be used to erase, upload or download the contents of flash or RAM binaries.  


#### Install Debugging Server

Get the `openocd` tool from the `xpm` package manager (and not the deprecated github releases page).
- Download Node.js [homepage](https://nodejs.org/en/download/)
- Install `openocd` for Windows from the latest maintainer, the `xpm` repository.
  ```batch
  npm install --global xpm
  xpm install --global @gnu-mcu-eclipse/arm-none-eabi-gcc
  ```
- Add it to your `PATH` variable, pick either method:  
  - Update Local PATH only:   
    ```batch
    SET PATH=%PATH%;%APPDATA%\xPacks\@xpack-dev-tools\openocd\0.10.0-13.1\.content\bin
    ```
  - Update Global PATH:
    - Open the "Environment Variables" menu by typing:  
      `<Win> + <r>`  
      `rundll32.exe sysdm.cpl,EditEnvironmentVariables`  
    - Edit/insert the PATH variable with the text:    
      ```batch
      %APPDATA%\xPacks\@xpack-dev-tools\openocd\0.10.0-13.1\.content\bin
      ```
- Verify  
  In a `cmd.exe` window execute the commands:
  ```batch
  openocd --version
  arm-none-eabi-gdb --version
  ```  


#### Install GDB Debugger  

Download the `gcc`/`gdb` tools for the ARM target of the STM32 Blue Pill.  
The GDB debugger is used because currently `lldb` used by Rust doesn't support the `load` command to flash the target device.
- Arm Build Tools [site](https://armkeil.blob.core.Windows.net/developer/Files/downloads/gnu-rm/8-2019q3/RC1.1/gcc-arm-none-eabi-8-2019-q3-update-win32.exe)


#### Install MSVC C++ Build Tools for Rust  

To use rust with the Windows native MSVC toolchain, we will need to install the MSVC C++ build tools dependency.
This **single** item can be obtained standalone from microsoft directly.
On the microsoft website the latest can be found as "Build Tools for Visual Studio 20xx" and navigate to download just the "Visual C++ Build Tools" component.

- MSVC C++ 2019 Build Tools [site](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019).    
  When the installer asks what sub-features of "Visual C++ Build Tools" are wanted, the defaults are fine.


#### Install Rust
Download and run the Windows installer `rustup-init.exe` from [rustup.rs](https://rustup.rs/).

After the install, from any command line add the Rust ARM target toolchain.
Rust assumes by default that you only need the native compiler, so we will explictly ask for the ARM cross-compiler.
```bash
rustup install stable # Installer default
rustup default stable # Installer default
rustup target add thumbv7m-none-eabi
```

#### Verify Rust Install

- Get the blinky LED example for STM32 blue pill from the `stm32f1xx-hal` repository.  
  ```batch
  git clone https://github.com/stm32-rs/stm32f1xx-hal
  cd stm32f1xx-hal
  ```

- Build the blink example program.  
  ```batch
  cargo clean
  cargo check
  cargo build --features stm32f103 --example blinky
  ```

- (Optional) Verify the build output.  
  ```batch
  arm-none-eabi-readelf -h target/thumbv7m-none-eabi/debug/examples/blinky
  ```

- Ensure the ST-Link v2 dongle is connected to both the computer and the Blue-Pill board.  
  Note, there are two variants of the 10-pin ST-Link v2 dongle (with "M" or "ST" logos), which have the same pins but slightly different pinout locations.  
  
| Blue-Pill SWD/JTAG | ST-Link v2 Dongle | 
| ------------- |-------------|
| 3V3 | 3.3V |
| SWIO | SWDIO |
| SWCLK | SWCLK |
| GND | GND |

- Attach `openocd` as the link between ST-LINK v2 probe and the `gdb` debugger.  
  ```batch
  openocd -f interface\stlink-v2.cfg -f target\stm32f1x.cfg
  ```  
  Leave this window running, as it will be your STDOUT if you are using semi-hosting in your code (allows printing to your screen over the debugger).

- Cargo (a Rust tool) requires a **one-time** config for allowing it permission to launch GDB for you.
  Issue this command from `cmd.exe` in the project folder, to allow `cargo` to launch and load the `gdb` debugger for you.
  ```batch
  echo set auto-load safe-path %CD% >> %USERPROFILE%\.gdbinit
  ```

- Load/run the program with `cargo`/`gdb`. (Use a build with debug info to allow breakpoints.)
  ```batch
  cargo run --features stm32f103 --example blinky
  ```


#### (Optional) Configure GDB for Cargo  

- Cargo (a Rust tool) requires a **one-time** config for allowing `cargo run` permission to launch GDB.
  Open `cmd.exe` and navigate to the project folder to run this command.
  ```batch
  cd <project_folder>
  echo set auto-load safe-path %CD% >> %USERPROFILE%\.gdbinit
  ```


#### (Optional) Manually Launch Debugger  

The debugger can manually load/run the STM32 Blue Pill from `gdb` after the binary has been built.  
This example assumes a `debug` build, but the same applies to `release` builds, just change the binary build folder.  
- Launch the debugger in the project folder.  
  ```batch
  arm-none-eabi-gdb target/thumbv7m-none-eabi/debug/examples/blinky
  ```

- Then issue these startup commands to connect and load the board.  
  ```gdb
  target remote :3333
  
  # print demangled symbols
  set print asm-demangle on
  
  # Break on unhandled exception, hard fault and panic handler functions
  break DefaultHandler
  break HardFault
  break rust_begin_unwind
  
  # Show semihosting/debug output
  monitor arm semihosting enable
  
  # Load the built binary to the Blue-Pill
  load
  
  # start the process but immediately halt the processor
  stepi
  ```  

- Common GDB commands [cheat-sheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf).  

- (Optional) GDB commands can be saved to a file (ex: `openocd_gdb_connect.gdb`) and run at startup.  
  ```batch
  arm-none-eabi-gdb -x openocd.gdb target/thumbv7m-none-eabi/debug/examples/blinky
  ```


------

### Tested Versions
| Binary | Reported Version | 
| ------------- | ------------- |
| `arm-none-eabi-gcc -v` | `gcc version 8.3.1 20190703 (release) [gcc-8-branch revision 273027] (GNU Tools for Arm Embedded Processors 8-2019-q3-update)` |
| `arm-none-eabi-gdb -v` | `GNU gdb (GNU Tools for Arm Embedded Processors 8-2019-q3-update) 8.3.0.20190703-git` |
| `arm-none-eabi-readelf --version` | `GNU readelf (GNU Tools for Arm Embedded Processors 8-2019-q3-update) 2.32.0.20190703` |
| `cargo --version` | `cargo 1.38.0 (23ef9a4ef 2019-08-20)` |
| `git --version` | `git version 2.23.0.Windows.1` |
| `npm --version` | `6.11.3` |
| `openocd --version` | `xPack OpenOCD, 64-bit Open On-Chip Debugger 0.10.0+dev (2019-07-17-11:28)` |
| `rustc --version` | `rustc 1.38.0 (625451e37 2019-09-23)` |
| `rustup --version` | `rustup 1.19.0 (2af131cf9 2019-09-08)` |
| `ST-LINK v2 Driver` | `2.1.0.0` |
| `xpm --version` | `0.5.0` |

------
