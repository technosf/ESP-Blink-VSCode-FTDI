# ESP-Blink-VSCode-FTDI
This repo contains the ESP32 Blink example with VSCode FTDI debug configs. It's where I'm experimenting with getting it all to work, which is somewhat problematic :(

<!-- TOC -->

- [Overview](#overview)
    - [Dev Board](#dev-board)
    - [FTDI Connector](#ftdi-connector)
    - [VSCode](#vscode)
        - [settings.json](#settingsjson)
        - [launch.jsonon](#launchjsonon)
- [Errors](#errors)
    - [ESP32 Monitor Output](#esp32-monitor-output)
    - [GDB Terminal](#gdb-terminal)
    - [OpenOCD Output](#openocd-output)
    - [VSCode](#vscode)

<!-- /TOC -->

## Overview
The idea of a debugger integrated with an IDE to poke around and step through the ESP32 is appealing, but the reality of getting it to work is challenging (in my experience).

VSCode provides an OpenOCD server (which communicates via an FTDI JTAG cable to the ESP32s [JTAG](https://en.wikipedia.org/wiki/JTAG) interface) and a generic GDB visual interface that is configured to use the ESP-IDFs `xtensa-esp32-elf-gdb` debugger. The debugger connects to OpenOCD and thus to the ESP32. _In theory._

In practice I appear to have the GDB terminal stepping through the code and the VSCode debugger stepping through something completely different.

### Dev Board
My dev system is a Intel-based Linux machine, with an ESP32 WROOM-32 devkit on `/dev/ttyESP`


### FTDI Connector
I'm using a [FTDI C232HM-DDHSL-0](https://www.ftdichip.com/Products/Cables/USBMPSSE.htm). It is _3.3v_ and it lives on `/dev/ttyFTDI`

### VSCode
I'm on the latest VSCode - _1.5.1.1_ as of writing.


#### settings.json
This is the OpenOCD server config that VSCode kicks off.
Note that I modified `interface/ftdi/c232hm.cfg` to add the comms speed with a line `adapter_khz 20000`.
```
  "idf.port": "/dev/ttyESP",
  "idf.showOnboardingOnInit": false,
  "idf.openOcdConfigs": [
      "/dev/ttyFTDI",
      "board/esp32-wrover-kit-3.3v.cfg",
      "interface/ftdi/c232hm.cfg"
  ]
  ```

#### launch.jsonon
  This is the GDB configuration.
  ```
   {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${env:HOME}/.espressif/tools/xtensa-esp32-elf/esp-2020r3-8.4.0/xtensa-esp32-elf/bin/xtensa-esp32-elf-gdb",
            "args": [
                "-x",
                "${workspaceFolder}/gdbinit",
                "${workspaceFolder}/build/blink.elf"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
  ```

## Errors
On the whole, it looks to all hang together, but trying to step through the code, hit break points or examine storage is non-functional.

### ESP32 Monitor Output
The chip hangs as if at a breakpoint waiting for VSCode to tell it to continue:
```
I (197) cpu_start: Application information:
I (201) cpu_start: Project name:     blink
I (206) cpu_start: App version:      1
I (211) cpu_start: Compile time:     Nov 14 2020 15:54:30
I (217) cpu_start: ELF file SHA256:  705499d79b77531d...
I (223) cpu_start: ESP-IDF:          v4.2-beta1-227-gf0e87c933
I (229) cpu_start: Starting app cpu, entry point is 0x400815c0
0x400815c0: call_start_cpu1 at ~/esp/esp-idf/components/esp32/cpu_start.c:287

I (221) cpu_start: App cpu up.
I (240) heap_init: Initializing. RAM available for dynamic allocation:
I (247) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (253) heap_init: At 3FFB28B0 len 0002D750 (181 KiB): DRAM
I (259) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (265) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (272) heap_init: At 40089EB0 len 00016150 (88 KiB): IRAM
I (278) cpu_start: Pro cpu start user code
I (296) spi_flash: detected chip: generic
I (297) spi_flash: flash io: dio
I (297) cpu_start: Starting scheduler on PRO CPU.
I (0) cpu_start: Starting scheduler on APP CPU.
```

### GDB Terminal
```
GNU gdb (crosstool-NG esp-2020r3) 8.1.0.20180627-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "--host=x86_64-build_pc-linux-gnu --target=xtensa-esp32-elf".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /home/martin/workspaces/vscode/ESP-Blink-VSCode-FTDI/build/blink.elf...done.
0x400e262a in esp_pm_impl_waiti () at /home/martin/esp/esp-idf/components/esp32/pm_esp32.c:484
484         asm("waiti 0");
JTAG tap: esp32.cpu0 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)
JTAG tap: esp32.cpu1 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)
cpu0: Debug controller 0 was reset.
cpu0: Core 0 was reset.
cpu0: Target halted, PC=0x500000CF, debug_reason=00000000
esp32: Core 0 was reset.
esp32: Debug controller 1 was reset.
esp32: Core 1 was reset.
Target halted. CPU0: PC=0x40000400 (active)
Target halted. CPU1: PC=0x40000400 
Hardware assisted breakpoint 1 at 0x400d2b07: file ../main/blink.c, line 28.
Target halted. CPU0: PC=0x400D2B07 (active)
Target halted. CPU1: PC=0x400E262A 
[New Thread 1073436320]
[New Thread 1073434428]
[New Thread 1073438968]
[New Thread 1073426644]
[New Thread 1073412788]
[New Thread 1073413520]
[New Thread 1073427784]
[Switching to Thread 1073432536]

Thread 1 hit Temporary breakpoint 1, app_main () at ../main/blink.c:28
28          gpio_pad_select_gpio(BLINK_GPIO);
(gdb) 
```

### OpenOCD Output
Looks good, AFAICT
```
Open On-Chip Debugger  v0.10.0-esp32-20200709 (2020-07-09-08:54)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : Configured 2 cores
Warn : Interface already configured, ignoring
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : ftdi: if you experience problems at higher adapter clocks, try the command "ftdi_tdo_sample_edge falling"
Info : clock speed 20000 kHz
Info : JTAG tap: esp32.cpu0 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)
Info : JTAG tap: esp32.cpu1 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)
Info : Target halted. CPU0: PC=0x400D2B07 (active)
Info : Target halted. CPU1: PC=0x400E262A 
Info : Listening on port 3333 for gdb connections
❌ Info : accepting 'gdb' connection on tcp/3333
Error: No symbols for FreeRTOS
Info : cpu0: Target halted, PC=0x40091856, debug_reason=00000001
Info : Flash mapping 0: 0x10020 -> 0x3f400020, 23 KB
Info : Flash mapping 1: 0x20020 -> 0x400d0020, 75 KB
Info : cpu0: Target halted, PC=0x40091856, debug_reason=00000001
Info : Auto-detected flash bank 'esp32.flash' size 4096 KB
Info : Using flash bank 'esp32.flash' size 4096 KB
Info : cpu0: Target halted, PC=0x40091856, debug_reason=00000001
Info : Flash mapping 0: 0x10020 -> 0x3f400020, 23 KB
Info : Flash mapping 1: 0x20020 -> 0x400d0020, 75 KB
Info : Using flash bank 'esp32.irom' size 76 KB
Info : cpu0: Target halted, PC=0x40091856, debug_reason=00000001
Info : Flash mapping 0: 0x10020 -> 0x3f400020, 23 KB
Info : Flash mapping 1: 0x20020 -> 0x400d0020, 75 KB
Info : Using flash bank 'esp32.drom' size 24 KB
Info : JTAG tap: esp32.cpu0 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)
Info : JTAG tap: esp32.cpu1 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)
Info : cpu0: Debug controller 0 was reset.
Info : cpu0: Core 0 was reset.
Info : cpu0: Target halted, PC=0x500000CF, debug_reason=00000000
Info : esp32: Core 0 was reset.
Info : esp32: Debug controller 1 was reset.
Info : esp32: Core 1 was reset.
Info : Target halted. CPU0: PC=0x40000400 (active)
Info : Target halted. CPU1: PC=0x40000400 
Info : Target halted. CPU0: PC=0x400D2B07 (active)
Info : Target halted. CPU1: PC=0x400E262A 
```

### VSCode
A couple of error panels come up trying to budge the debugger, they complain of missing source:
```
Unable to open 'poll.c': Unable to read file '/build/glibc-ZN95T4/glibc-2.31/sysdeps/unix/sysv/linux/poll.c' (Error: Unable to resolve non-existing file '/build/glibc-ZN95T4/glibc-2.31/sysdeps/unix/sysv/linux/poll.c').
```

```
Unable to open 'libc-start.c': Unable to read file '/build/glibc-ZN95T4/glibc-2.31/csu/libc-start.c' (Error: Unable to resolve non-existing file '/build/glibc-ZN95T4/glibc-2.31/csu/libc-start.c').
```
