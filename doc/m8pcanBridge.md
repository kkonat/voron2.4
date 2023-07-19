# Reflashing Manta M8P to can bridge mode

```cd klipper
make clean
make menuconfig
```

Important thing is to set the USB to CAN Bridge PA11/PA12 and CAN bus interface PD12/PD13

```Micro-controller Architecture (STMicroelectronics STM32)  --->
    Processor model (STM32G0B1)  --->
    Bootloader offset (8KiB bootloader)  --->
    Clock Reference (8 MHz crystal)  --->
    Communication interface (USB to CAN bus bridge (USB on PA11/PA12))  --->
    CAN bus interface (CAN bus (on PD12/PD13))  --->
    USB ids  --->
(1000000) CAN bus speed
```

and of course the same CAN bus speed as in every other device

quit menuconfig by pressing 'Q' and confirming 'Y' for yes

Then build the image
`make`

switch Manta M8P to DFU mode by pressing and holding BOOT0 & RESET, and flash the just made klipper image using:
`make flash FLASH_DEVICE=0483:df11`

It should display uploading progress:

```
Determining device status: state = dfuIDLE, status = 0
dfuIDLE, continuing
DFU mode device DFU version 011a
Device returned transfer size 1024
DfuSe interface name: "Internal Flash   "
Downloading to address = 0x08002000, size = 29824
Download        [=========================] 100%        29824 bytes
Download done.
File downloaded successfully
```

And immediatelly after that it errors out like that...

```
dfu-util: Error during download get_status

Failed to flash to 0483:df11: Error running dfu-util

If the device is already in bootloader mode it can be flashed with the
following command:
  make flash FLASH_DEVICE=0483:df11
  OR
  make flash FLASH_DEVICE=1209:beba

If attempting to flash via 3.3V serial, then use:
  make serialflash FLASH_DEVICE=0483:df11

make: *** [src/stm32/Makefile:111: flash] Error 255
```

But don't worry. This is ok. One line above it said it downloaded the file succesfully. Then it left the DFU mode right away so the latter part errored.

You may now check for the USB canbus bridge using
`lsusb`
one of the devices listed should be
`Bus 002 Device 005: ID 1d50:606f OpenMoko, Inc. Geschwister Schneider CAN adapter`
