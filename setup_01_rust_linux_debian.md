# STM32 Blue Pill Setup

## Linux Environment Setup

### Prerequisites

These instructions are mainly targeted at Debian/Ubuntu/Mint (apt package manager) based linux distros, and assume nothing is yet installed (fresh install).
The majority of the setup is to accommodate the tools which interact with the non-native HW platform.

#### BuildUtils Install
```bash
sudo apt install pkg-config cmake libssl-dev zlib1g-dev gdb-multiarch curl git
sudo ln -s /usr/bin/gdb-multiarch /usr/bin/arm-none-eabi-gdb

sudo apt install binutils-arm-none-eabi gcc-arm-none-eabi
```

Open new terminal window to verify it works.
```bash
arm-none-eabi-gcc -v
```

#### Debugger Install
```bash
sudo apt install openocd
```

#### Rust Install

Get `rustup` and follow install instructions.
``` bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After the install, from any command line install the Rust ARM target toolchain.
Rust assumes by default that you only need the native compiler, so we will explictly ask for the ARM cross-compiler.
```bash
rustup install stable # Installer default
rustup default stable # Installer default
rustup target add thumbv7m-none-eabi
```

Reboot to update your terminal environment script with the Rust tools path, or issue the command the installer mentions to get started now.
```bash
source $HOME/.cargo/env
```

#### Identify USB Dongle

At this point, the USB probe/dongle should be detected by the linux machine.
This can be double checked by performing a diff between when the debugger is connected and disconnected.
```bash
lsusb -v > 0.txt
echo Remove or Attach the USB dongle now
sleep 30
lsusb -v > 1.txt
diff 0.txt 1.txt
```

------

### Build Example

#### Clone Example
Get the blinky LED example for STM32 blue pill from the `stm32f1xx-hal` repository.

```bash
git clone https://github.com/stm32-rs/stm32f1xx-hal
cd stm32f1xx-hal
```

#### Cargo Build
Build the blink program for the target processor `stm32f103` (an option in the stm32f1xx-hal repository we are using for this example).
```bash
cargo clean
cargo check
cargo build --features stm32f103 --example blinky
```

#### Launch Example

##### Verify Binary
Verify the build output.
```bash
arm-none-eabi-readelf -h target/thumbv7m-none-eabi/debug/examples/blinky
```

##### Attach Debugger Server
The below command request launches `openocd` as the link between ST-LINK v2 probe and the debugger.
```bash
openocd -f interface/stlink-v2.cfg -f target/stm32f1x.cfg
```
Leave this window running, as it will be your STDOUT if you are using semi-hosting in your code (allows printing to your screen over the debugger).

###### One Time Config

Allow cargo to launch and load the gdb debugger for you.
```bash
echo "set auto-load safe-path $(pwd)" >> ~/.gdbinit 
``` 

Or optionally allow for all folders/projects.
```bash
echo "set auto-load safe-path /" >> ~/.gdbinit 
```

##### Cargo Run
Load/run the program with `cargo`/`gdb`. (Use a build with debug info to allow breakpoints.)
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

## GDB Debugger Commands
See the gdb [cheat-sheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf) for usage details.

## Tested Versions
| Binary | Reported Version | 
| ------------- | ------------- |
| `arm-none-eabi-gcc -v` | `gcc version 6.3.1 20170620 (15:6.3.1+svn253039-1build1)` |
| `arm-none-eabi-gdb -v` | `GNU gdb (Ubuntu 8.1-0ubuntu3.1) 8.1.0.2180409-git` |
| `arm-none-eabi-readelf --version` | `GNU readelf (2.27-9ubuntu1+9) 2.27` |
| `cargo --version` | `cargo 1.38.0 (23ef9a4ef 2019-08-20)` |
| `git --version` | `git version 2.17.1` |
| `openocd --version` | `Open On-Chip Debugger 0.10.0` |
| `rustc --version` | `rustc 1.38.0 (625451e37 2019-09-23)` |
| `rustup --version` | `rustup 1.19.0 (2af131cf9 2019-09-08)` |

------
