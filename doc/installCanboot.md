You build need to flash [CANBOOT](https://github.com/Arksine/CanBoot) image to the SB2209 CAN board, only if you want to upgrade the firmware via CAN later.

After you flash this image, you will no longer be able to flash the firmware over USB - over usb, SB2209 will not be visible anymore

As described in [SB2209 CAN Build manual](<https://github.com/bigtreetech/EBB/blob/master/EBB%20SB2209%20CAN%20(RP2040)/Build%20Guide/EBB%20SB2209%20CAN%20V1.0%EF%BC%88RP2040%EF%BC%89BUILD%20GUIDE.pdf>) page 25

BUILDING & FLASHING CANBOOT

```cd ~
git clone https://github.com/Arksine/CanBoot
cd CanBoot
make menuconfig
```

Use these settings:

```
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32G0B1)
    Build CanBoot deployment application (Do not build)
    Clock Reference (8 MHz crystal)
    Communication interface (CAN bus (on PB0/PB1))
    Application start offset (8KiB offset)
    (1000000) CAN bus speed

    () GPIO pins to set on bootloader entry
    [*] Support bootloader entry on rapid double click of reset button
    [ ] Enable bootloader entry on button (or gpio) state
    [*] Enable Status LED
    (PA13) Status LED GPIO Pi

```

then
`make`

then
press and hould BOOT and then press RESET on SB2209 to go to the DFU boot mode

On CB1 list USB devices:
`lsusb`
you should see:

```
Bus 001 Device 005: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
Bus 001 Device 004: ID 1d50:6061 OpenMoko, Inc. Geschwister Schneider CAN adapter
Bus 001 Device 003: ID 0424:0c00 Microchip Technology, Inc. ( formerly SMSC ) SMC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Microchip Technology, Inc. ( formerly SMSC ) SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

This 0483:df11 is the id of the SB2209 USB

You flash canboot to SB2209 using:
`sudo dfu-util -a 0 -d 0483:df11 -s 0x08000000:mass-erase:force -D ~/CanBoot/out/canboot.bin
`
After it is flashed the red LED will blink in 1 second interval (before it was dimly glowing red)

Now you have SB2209:

- flashed with canboot
- set to communicate over CAN (PB0/PB1)
- set the CAN clock speed to 1000000
