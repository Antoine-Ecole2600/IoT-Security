# IoT Security – Reverse Challenges

This document provides a comprehensive overview of the reverse engineering challenges encountered during the IoT Security course, with a focus on analyzing and interacting with emulated firmware in an AVR environment.

## Environment Setup

For these challenges, we are provided with the following files :

- `run.sh`  
- `firmware_debug.elf`

The `firmware_debug.elf` file is the target firmware that we need to reverse engineer in order to extract the flag.

The `run.sh` script contains the following command used to launch a QEMU instance emulating an AVR microcontroller :

```bash
qemu-system-avr -M mega2560 -bios firmware_debug.elf -nographic -serial tcp::5678,server=on,wait=off -s -S "$@"
```

Below is a detailed breakdown of the command and its components :

## Command Breakdown

- `qemu-system-avr`  
  Launches QEMU in AVR emulation mode. QEMU is a hardware emulator capable of simulating various processor architectures. In this case, it emulates an **AVR microcontroller**, a widely used 8-bit architecture (notably found in Arduino boards).

- `-M mega2560`  
  Specifies the machine to emulate. The `mega2560` refers to the **Arduino Mega 2560**, which is based on the ATmega2560 microcontroller.

- `-bios firmware_debug.elf`  
  Loads the specified firmware (`firmware_debug.elf`) into the emulated device. The `.elf` format contains the binary code along with debug information, making it suitable for reverse engineering.

- `-nographic`  
  Disables the graphical interface. All input/output (e.g., serial communication) is redirected to the terminal. This is useful for working in headless environments or scripts.

- `-serial tcp::5678,server=on,wait=off`  
  Redirects the AVR's serial output to a TCP server on port `5678`.  
  - `server=on` configures QEMU as a TCP server.  
  - `wait=off` allows QEMU to start immediately, without waiting for a client to connect.  
  This setup enables external tools (like `netcat`) to interact with the serial interface over TCP.

- `-s`  
  Starts a GDB server on port `1234`. This allows remote debugging using tools such as `avr-gdb`, making it possible to inspect memory, set breakpoints, and control execution.

- `-S`  
  Prevents the CPU from starting automatically. The firmware remains paused at startup, giving you the opportunity to attach a debugger and set initial breakpoints before execution begins.

- `"$@"`  
  Shell syntax that expands to any additional arguments passed to the script. This provides flexibility to pass custom QEMU options when launching the script.

## Solution

For all the challenges, we assume the flag format follows the convention : `IOT{...}`.

### Level 1

As a first step in the analysis, we use the `strings` command to extract printable strings from the `.elf` file and filter for any that match the expected flag format.

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

### Level 2

For this level, I initially attempted to use the `strings` command on the `.elf` file, but unfortunately, it did not yield any useful information.

Next, I connected to the QEMU instance using `avr-gdb` to perform a more detailed analysis of the firmware, allowing me to inspect memory, variables, and functions in order to locate the flag.

#### Connecting to QEMU via GDB

To begin, I connected to the QEMU machine using the following command in GDB :

```bash
(gdb) target remote localhost:1234

Remote debugging using localhost:1234
```

#### Inspecting Functions

Once connected, I listed all the available functions by using the `info functions` command. This command provides a list of functions that have been defined within the firmware, and could potentially contain code that is useful for locating the flag.

```bash
(gdb) info functions

All defined functions:

File main.c:
14:     void get_input(void);
40:     int main(void);

File uart.c:
31:     void uart_flush(void);
26:     char uart_get(void);
10:     void uart_init(void);
38:     void uart_putc(char);
18:     void uart_puts(char *);
43:     void uart_puts_p(const char *);

[...]
```

From the list of functions, none seemed directly relevant to the flag retrieval process, so I shifted my focus to the `main` function and the global variables defined there, as they were more likely to contain key information.

#### Inspecting Variables

Therefor, I listed the global variables in `main.c` using the `info variables` command. This command lists all the variables in the code, and I was looking for any that might hold the flag or other crucial data.

```bash
(gdb) info variables

File main.c:
2:      const char key[70];
1:      char secret[70];

[...]
```

Two variables were of particular interest :

- `key`
- `secret`

To explore the contents of these variables, I used the `print` command in GDB.

```bash
(gdb) print key

$1 = "\270pbKQ\242\331\362\242\305\301\006\352\221F\367\235\320\025\301\221\273\360\326\022\3576L\264\274\246\237\b\305t+\237\243N\266<\004\352G_ϝ\210G\317\374u\231\215\222\312\375\\\344\033˗xHտ\270", <incomplete sequence \317>

(gdb) print secret

$2 = "IOT{3c272d7cb61ca4a6041418d5a02901dd2ee0e9b93be330df3e21c729a79a0178}"
```

#### Conclusion

After inspecting the code and variables, I found the flag stored in the `secret` variable. The flag for this level is :

```
IOT{3c272d7cb61ca4a6041418d5a02901dd2ee0e9b93be330df3e21c729a79a0178}
```

This suggests that the flag was hardcoded in the firmware within the secret variable. To retrieve it, I had to connect to the firmware using GDB, inspect the variables, and extract the flag manually.
