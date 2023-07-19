# Preparing klipper image for sb2209 toolhead and flashing it over CAN

### 1. First prepare the image

```
cd ~/klipeer
make clean
make menuconfig
```

use these settings (SB 2209 specific):

```
Klipper Firmware Configuration
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)  --->
    Processor model (STM32G0B1)  --->
    Bootloader offset (8KiB bootloader)  --->
    Clock Reference (8 MHz crystal)  --->
    Communication interface (CAN bus (on PB0/PB1))  --->
(1000000) CAN bus speed
()  GPIO pins to set at micro-controller startup
```

Important:

- you need to add the Bootloader offset (so flashing doesn't overwrite the bootloader)
- set communication interface to CAN bus on PB0/PB1 (it's different than on MANTA)
- set CAN bus speed to the same speed as everywhere

### 2. Flash it over CAN bus using CanBoot:

`python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u 3b1b4f8763a2`
where 3b1b4f8763a2 is your canbus_uuid of the SB2009

it should output something like:

```
Sending bootloader jump command...
Resetting all bootloader node IDs...
Checking for canboot nodes...
Detected UUID: a70885e01a2a, Application: Klipper
Detected UUID: 3b1b4f8763a2, Application: CanBoot
Attempting to connect to bootloader
CanBoot Connected
Protocol Version: 1.0.0
Block Size: 64 bytes
Application Start: 0x8002000
MCU type: stm32g0b1xx
Verifying canbus connection
Flashing '/home/biqu/klipper/out/klipper.bin'...

[##################################################]

Write complete: 14 pages
Verifying (block count = 434)...

[##################################################]

Verification Complete: SHA = 4ACE998420DD45FC79427A666851D6B7A2661AAA
CAN Flash Success
```

Yuo can re-flash the new firmware in the same way

**warning**: if you are re-flashing, with the canbus_uuid already entered in the printer.cfg, flashing will hang up. So before flashing, you'll have to stop "Klipper-mcu" from the top-right "power button" menu in mainsail.

### 3. Check

use
`~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`
to see what can bus devices are _available_ (if they are consumed by Klipper, i.e. entered into the printer.cfg file, they will not show up here, but they will apper in klipper)
