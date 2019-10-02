# STM32 Blue Pill Setup

## Windows Environment Setup

### Prerequisites

These instructions are targeted at Windows 7+ (tested on windows 10), and assume nothing is yet installed (fresh install).
The majority of the setup is to accommodate the tools which interact with the non-native HW platform.

Required prerequisites:
#### Git for Windows
Get `git` so the example code can be downloaded from github, and you can manage version control in your own projects.
- Git for windows [homepage](https://gitforwindows.org/)

#### ARM Tools

##### Cross-Compiler Install
Get the `gcc`/`gdb` tools for the ARM target of the STM32 Blue Pill.
- Arm Build Tools [site](https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/8-2019q3/RC1.1/gcc-arm-none-eabi-8-2019-q3-update-win32.exe)

##### ST-LINK v2 Driver
The manufacturer hides the ST-LINK v2 driver behind a free account requiring email for registration.
So first register for a free account, then download the driver (no extra tools, only the windows driver is required).
http://www.st.com/en/embedded-software/stsw-link009.html

##### Debugger Install
Get the `openocd` tool from the `xpm` package manager (and not the deprecated github page).
- Download Node.js [homepage](https://nodejs.org/en/download/)
- Install `openocd` for windows from the latest maintainer, the `xpm` repository.
  ```batch
  npm install --global xpm
  xpm install --global @gnu-mcu-eclipse/arm-none-eabi-gcc
  ```
- Add it to your `PATH` variable.    
  Example:    
  ```batch
  SET PATH=%PATH%;%APPDATA%\xPacks\@xpack-dev-tools\openocd\0.10.0-13.1\.content\bin
  ```
  Or perminately add it to your path by launching the PATH edit menu.    
  - `<Win> + <r>`
  - Enter `rundll32.exe sysdm.cpl,EditEnvironmentVariables`
  - Edit/add an entry into the PATH variable containing:    
    ```batch
	%APPDATA%\xPacks\@xpack-dev-tools\openocd\0.10.0-13.1\.content\bin
	```

#### Rust Install

##### MSVC C++ Build Tools Install
If we want to use the windows native MSVC toolchain, we will need to install the MSVC C++ build tools dependency.
This **single** item can be obtained standalone from microsoft directly.
We will get it from "Build Tools for Visual Studio 20xx" and navigate to download just the "Visual C++ Build Tools" on the website.

- MSVC C++ 2019 Build Tools [site](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019).    
  When the installer asks what sub-features of "Visual C++ Build Tools" are wanted, the defaults are fine.

##### Rust Installer
Download and run the windows installer `rustup-init.exe` from [rustup.rs](https://rustup.rs/).

After the install, from any command line install the Rust ARM target toolchain.
Rust assumes by default that you only need the native compiler, so we will explictly ask for the ARM cross-compiler.
```bash
rustup install stable # Installer default
rustup default stable # Installer default
rustup target add thumbv7m-none-eabi
```

------

### Build Example

#### Clone Example
Get the blinky LED example for STM32 blue pill from the `stm32f1xx-hal` repository.

```batch
git clone https://github.com/stm32-rs/stm32f1xx-hal
cd stm32f1xx-hal
```

#### Cargo Build
Build the blink program
```batch
cargo clean
cargo check
cargo build --features stm32f103 --example blinky
```

#### Launch Example

##### Verify Binary
Verify the build output.
```batch
arm-none-eabi-readelf -h target/thumbv7m-none-eabi/debug/examples/blinky
```
##### Attach Debugger Server
The below windows command line request launches `openocd` as the link between ST-LINK v2 probe and the `gdb` debugger.
```batch
openocd -f interface\stlink-v2.cfg -f target\stm32f1x.cfg
```
Leave this window running, as it will be your STDOUT if you are using semi-hosting in your code (allows printing to your screen over the debugger).

##### Cargo Launch Debugger

###### One Time Config
Issue this command from a windows `cmd.exe` command line to allow cargo to launch and load the gdb debugger for you.
```batch
echo set auto-load safe-path %CD% >> %USERPROFILE%\.gdbinit
```

###### Cargo Run
Load/run the program with `cargo`/`gdb`. (Only non-release will allow proper breakpoints.)
```batch
cargo run --features stm32f103 --example blinky
```

##### Manual Launch Debugger
Optionally you can manually load/run the STM32 Blue Pill from gdb after it has been built.
This example assumes a `debug` build, but the same applies to `release` builds, just change the binary build folder.
Launch the debugger.
```bash
arm-none-eabi-gdb target/thumbv7m-none-eabi/debug/examples/blinky
```

Then issue these startup commands to connect and load the board.
```gdb
target remote :3333

 # print demangled symbols
set print asm-demangle on

 # detect unhandled exceptions, hard faults and panics
break DefaultHandler
break HardFault
break rust_begin_unwind

monitor arm semihosting enable

load

 # start the process but immediately halt the processor
stepi
```

Or optionally save them to a file `openocd.gdb` and launch the debugger like this:
```bash
arm-none-eabi-gdb -x openocd.gdb target/thumbv7m-none-eabi/debug/examples/blinky
```

------

### GDB Debugger Commands
See the gdb [cheat-sheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf) for usage details.

------

### Tested Versions
| Binary | Reported Version | 
| ------------- | ------------- |
| `arm-none-eabi-gcc -v` | `gcc version 8.3.1 20190703 (release) [gcc-8-branch revision 273027] (GNU Tools for Arm Embedded Processors 8-2019-q3-update)` |
| `arm-none-eabi-gdb -v` | `GNU gdb (GNU Tools for Arm Embedded Processors 8-2019-q3-update) 8.3.0.20190703-git` |
| `arm-none-eabi-readelf --version` | `GNU readelf (GNU Tools for Arm Embedded Processors 8-2019-q3-update) 2.32.0.20190703` |
| `cargo --version` | `cargo 1.38.0 (23ef9a4ef 2019-08-20)` |
| `git --version` | `git version 2.23.0.windows.1` |
| `npm --version` | `6.11.3` |
| `openocd --version` | `xPack OpenOCD, 64-bit Open On-Chip Debugger 0.10.0+dev (2019-07-17-11:28)` |
| `rustc --version` | `rustc 1.38.0 (625451e37 2019-09-23)` |
| `rustup --version` | `rustup 1.19.0 (2af131cf9 2019-09-08)` |
| `ST-LINK v2 Driver` | `2.1.0.0` |
| `xpm --version` | `0.5.0` |

------
