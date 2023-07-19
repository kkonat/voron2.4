# Setting up Voron 24. with BTT Manta 8mp + CB1 + BTT SB2240 Canbus

## 1. Hardware:

### Controller:

**Big Tree Tech Manta m8p v1.1**
[github.com/bigtreetech/manta-m8p](https://github.com/bigtreetech/manta-m8p)
Manual: [BIGTREETECH MANTA M8P V1.0&V1.1 User Manual.pdf](https://github.com/bigtreetech/Manta-M8P/blob/master/BIGTREETECH%20MANTA%20M8P%20V1.0%26V1.1%20User%20Manual.pdf)

### SoC module:

**CB1**
[github.com/bigtreetech/CB1](https://github.com/bigtreetech/CB1)
Manual: [BIGTREETECH CB1 User Manual.pdf](https://github.com/bigtreetech/CB1/blob/master/BIGTREETECH%20CB1%20User%20Manual.pdf)

### Canbus toolhead

**Big Tree Tech SB2209 Can v1.0**
github: [https://github.com/bigtreetech/EBB](https://github.com/bigtreetech/EBB/)
Manual: [EBB SB2240 2209 CAN v1.0 Build Guide.pdf](<https://github.com/bigtreetech/EBB/blob/master/EBB%20SB2209%20CAN%20(RP2040)/Build%20Guide/EBB%20SB2209%20CAN%20V1.0%EF%BC%88RP2040%EF%BC%89BUILD%20GUIDE.pdf>)
The procedure so far:

- [Install linux on CB1 and Klipper on M8P ](doc/installKlipper.md)

- [Install canboot on SB2209 Can](doc/installCanboot.md) (optional - only if you need to upload firmware over CAN, otherwise USB will be used for that)

- Connect SB2209 and M8P with CAN cable, enable terminators and power supply

  - CAN headers on Manta M8P 1.1 are the same - they only have CAN L / CAN H lines (PD12 / PD13) - so it doesn't matter to which you connect
  - I keep SB2209 supplied from USB port (jumper on USB_5V)
  - Connect only CAN Hi/Lo cable lnes between manta and SB2209
  - 120 Ohm Terminators are needed on both ends (m8p SB2209) - use right jumpers for this

- [Reflash Manta M8P to workin in the CAN bridge mode](doc/m8pcanBridge.md)

After flashing check which CAN bus devices are available

`~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`
you should see something like this:

```
Found canbus_uuid=a70885e01a2a, Application: Klipper
Found canbus_uuid=3b1b4f8763a2, Application: CanBoot
```

The Klipper device is Manta M8P just flashed
The CanBoot device is my SB 2209 CAN

If a davice says "Application: CanBoot" it means only the bootloader is responding and you have to program the klipper image in it

So in my case SB 2209 needs klipper.img
so prepare klipper image for it and flash SB 2209 over can bus do:

```
cd ~/klipeer
make clean
make menuconfig
```

use these settings (SB 2209 specific)

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

- you need to add the Bootloader offset (so it doesn't overwrite the bootloader)
- set communication interface to CAN bus on PB0/PB1
- set CAN bus speed to the same speed

now flash it over CAN bus using CanBoot:
`python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u 3b1b4f8763a2`
where 3b1b4f8763a2 is your canbus_uuid of the SB 2009

warning: if you are re-flashing while the canbus_uuid is entered in the printer.cfg, flashing will fail. So before that you have to stop "Klipper-mcu" from the top-right "power button" menu in mainsail

---

Helpful videos:

[Bigtreetech 必趣 Manta M8P v1.1 + EBB Canbus CanBoot Klipper 完整教程](https://www.youtube.com/watch?v=ekbxtDS_8cM&t=327s) by Botio 波提歐, in Chineese, very fast, but can be followed by looking at screens
It shows how to:

- install Klipper
- use Klipper Installation And Update Helper [kiauh](https://github.com/th33xitus/kiauh) to update your installation (he selects action 2 "Update")
- build CanBoot bootloader for M8P (canboot.bin)

        Build CanBoot deployment application (Do not build)
        Communication interface (CAN bus (on PD12/PD13))

- build Klipper fw for M8P in USB CAN bridge mode (klipper.bin)

        Communication interface (USB to CAN bus bridge (USB on PA11/PA12))
        CAN bus interface (CAN bus (on PD12/PD13))

- flash canboot.bin (to M8P i.e. 0483:df11)
  `sudo dfu-util -a 0 -D ~/CanBoot/out/canboot.bin --dfuse-address 0x8000000:force:mass-erase -d 0483:df11`

- flash klipper.bin also to 0483:df11, but to different address (0x8002000)
  `sudo dfu-util -a 0 -d 0483:df11 -D ~/klipper/out/klipper.bin --dfuse-address 0x8002000:force:mass-erase`

- enable can0 interface
  `sudo nano /ec/network/interfaces.d/can0`
  containing
  ```
  allow-hotplug can0
   iface can0 can static
   bitrate 1000000
   up ifconfig $IFACE txqueuelen 1024
  ```

```
 he uses 250000 in the video not 1000000

- query canbus
 `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`
 `
 to get canbus_uuid
 so it can be put into **printer.cfg**

```

```
[mcu]
canbus_uuid:001122344
#serial : /dev/serial/by-id/<your-mcu-id>

```

as soon as it is added to the printer.cfg it will disappear from the canbus_query list

He then goes connects a Can toolhead
and again performs canbus_quary on can0 to find the other device's canbus_uuid
which he adds to the printer.cfg in [mcu ebb36] section

After this both canbus devices are visible in Klipper but not visible in query

[Manta M8P+EBB CANbus Setup](https://www.youtube.com/watch?v=EA-oBfenxAE) by Stacking Layers
this is slower

```

```

```

```
