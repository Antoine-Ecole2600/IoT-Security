# IoT Security – Reverse Challenges

This document provides a comprehensive overview of the reverse engineering challenges encountered during the IoT Security course, with a focus on analyzing and interacting with emulated firmware in an AVR environment.

## Environment Setup

For these challenges, we are provided with the following files :

- `run.sh`  
- `firmware_debug.elf`

The `firmware_debug.elf` file is the target firmware that we need to reverse engineer in order to extract the flag.

The `run.sh` script contains the command used to launch a QEMU instance emulating an AVR microcontroller :

```bash
qemu-system-avr -M mega2560 -bios firmware_debug.elf -nographic -serial tcp::5678,server=on,wait=off -s -S "$@"
```

Below is a detailed breakdown of the command and its components :

## Command Breakdown

- **`qemu-system-avr`**  
  Launches QEMU in AVR emulation mode. QEMU is a hardware emulator capable of simulating various processor architectures. In this case, it emulates an **AVR microcontroller**, a widely used 8-bit architecture (notably found in Arduino boards).

- **`-M mega2560`**  
  Specifies the machine to emulate. The `mega2560` refers to the **Arduino Mega 2560**, which is based on the ATmega2560 microcontroller.

- **`-bios firmware_debug.elf`**  
  Loads the specified firmware (`firmware_debug.elf`) into the emulated device. The `.elf` format contains the binary code along with debug information, making it suitable for reverse engineering.

- **`-nographic`**  
  Disables the graphical interface. All input/output (e.g., serial communication) is redirected to the terminal. This is useful for working in headless environments or scripts.

- **`-serial tcp::5678,server=on,wait=off`**  
  Redirects the AVR's serial output to a TCP server on port `5678`.  
  - `server=on` configures QEMU as a TCP server.  
  - `wait=off` allows QEMU to start immediately, without waiting for a client to connect.  
  This setup enables external tools (like `netcat`) to interact with the serial interface over TCP.

- **`-s`**  
  Starts a GDB server on port `1234`. This allows remote debugging using tools such as `avr-gdb`, making it possible to inspect memory, set breakpoints, and control execution.

- **`-S`**  
  Prevents the CPU from starting automatically. The firmware remains paused at startup, giving you the opportunity to attach a debugger and set initial breakpoints before execution begins.

- **`"$@"`**  
  Shell syntax that expands to any additional arguments passed to the script. This provides flexibility to pass custom QEMU options when launching the script.

## Solution

For all the challenges, we assume the flag format follows the convention : `IOT{...}`.

### Level 1

As a first step in the analysis, we use the `strings` command to extract printable strings from the `.elf` file and filter for any that match the expected flag format :

```bash
$ strings firmware_debug.elf | grep -i "IOT"

IOT{97965bc48c105d11a8d5c4cec60b3312cc261440ed9fcb91b2f21949f7d862c9}
/home/user/Documents/Perso/IOT/reverse/lvl1/
```

From the output above, we can clearly identify the flag :

```
IOT{97965bc48c105d11a8d5c4cec60b3312cc261440ed9fcb91b2f21949f7d862c9}
```

This suggests that the flag was stored as a hardcoded string in the firmware, and `strings` was sufficient to locate it without requiring execution or disassembly.
