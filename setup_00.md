# STM32 Blue Pill Notes

## Environment Setup

There are a few common environments which can be used with the STM32F103 series Blue Pill, and which can generally be applied to other boards in the STM32Fxxx family of ARM chips.  

* Plain GCC (C/C++)
* Arduino IDE (C/C++)
* Rust

Each of these environments usually provide a toolchain with an ARM compiler, flasher, and (optional) debugger.

#### Required Hardware

* STM32 Blue-Pill
* ST-Link v2

#### Plain C/C++ Environment

Plain C/C++ on Linux (TODO)  
Plain C/C++ on Windows (TODO)  

* C/C++ Compiler  -  GCC
* On-Chip Debugging Server  -  OpenOCD
* Flasher  -  GDB
* Debugger  -  GDB
  
#### Arduino IDE Environment  

Arudino IDE on Linux (TODO)  
[Arduino IDE on Windows](setup_01_arduino_windows.md)  

* C/C++ Compiler  -  Arduino IDE (GCC)
* On-Chip Debugging Server  -  (Optional) OpenOCD
* Flasher  -  ST-Link / (Optional) GDB
* Debugger  -  (Optional) GDB

Notes:
* The Arudino IDE can also flash over serial using an optional bootloader (not described here).
* Arduino instructions are based on the [One Transistor](https://www.onetransistor.eu/2017/11/stm32-bluepill-arduino-ide.html) article.

#### Rust Environment

[Rust on Debian based Linux](setup_01_rust_linux_debian.md)  
[Rust on Windows](setup_01_rust_windows.md)  

* Rust Compiler  -  rustc
* On-Chip Debugging Server  -  OpenOCD
* Flasher  -  GDB
* Debugger  -  Cargo/GDB

Based on the articles:
* [STM32 Blue-Pill Rust](https://github.com/lupyuen/stm32-blue-pill-rust)  
* [STM32 Blue-Pill and Visual Studio Code](https://medium.com/coinmonks/coding-the-stm32-blue-pill-with-rust-and-visual-studio-code-b21615d8a20)  

