# Setting up Voron 2.4 with BTT Manta M8P + CB1 + BTT 2209/2240 for can communications

---

### HARDWARE:

#### 1. Controller:

**Big Tree Tech Manta m8p v1.1**
[github.com/bigtreetech/manta-m8p](https://github.com/bigtreetech/manta-m8p)
Manual: [BIGTREETECH MANTA M8P V1.0&V1.1 User Manual.pdf](https://github.com/bigtreetech/Manta-M8P/blob/master/BIGTREETECH%20MANTA%20M8P%20V1.0%26V1.1%20User%20Manual.pdf)

#### 2. SoC module:

**CB1** BTT's cheaper alternative or Raspberry CM4 compute module
[github.com/bigtreetech/CB1](https://github.com/bigtreetech/CB1)
Manual: [BIGTREETECH CB1 User Manual.pdf](https://github.com/bigtreetech/CB1/blob/master/BIGTREETECH%20CB1%20User%20Manual.pdf)

#### 3. Canbus toolhead

**Big Tree Tech SB2209 Can v1.0**
github: [https://github.com/bigtreetech/EBB](https://github.com/bigtreetech/EBB/)
Manual: [EBB SB2240 2209 CAN v1.0 Build Guide.pdf](<https://github.com/bigtreetech/EBB/blob/master/EBB%20SB2209%20CAN%20(RP2040)/Build%20Guide/EBB%20SB2209%20CAN%20V1.0%EF%BC%88RP2040%EF%BC%89BUILD%20GUIDE.pdf>)

SB2240 & SB2209 are basically the same. The only difference is that "the EBB SB2240 CAN uses SPI mode, whereas the EBB SB2209 CAN uses UART" for motor drive.

---

## Procedure

It describes how to program and test everything right on your desktop, before you install the gear in printer. Basically the Manta M8P is powered via USB (remember to plug the jumper) and the SB2209 toolhead is powered via USB (also needs jumper) cable plugged into Manta M8P.
So with +5V and GND over USB you only need to send two CAN bus signals between the boards.

### 1. [Install linux on CB1 and Klipper on M8P ](doc/installKlipper.md)

### 2. [Install canboot on SB2209 Can](doc/installCanboot.md)

This is needed, if you want to upload SB 2209 firmware over CAN later, otherwise you may keep using USB (which is the default factory setting) for that. Yet CanBoot is more convenient. You will need not to disassemble the printhead to get to BOOT0 RESET buttons. Firmware can be sent using a command line.

### 3. Connect SB2209 and M8P with CAN cable, enable terminators and provide power supply

- There are 2 black 2-wire CAN headers on Manta M8P 1.1. They are the same - they only have CAN L / CAN H lines (PD12 / PD13) - so it doesn't matter to which you connect.
- For testing you may keep SB2209 supplied from the USB port (jumper on USB_5V)
- So you will only need to connect just CAN Hi/Lo cable lnes between MANTA and SB2209
- 120 Ohm Terminators are needed on **both ends** (m8p SB2209) - use right jumpers for this

### 4. [Reflash Manta M8P to workin in the CAN bridge mode](doc/m8pcanBridge.md)

After flashing check which CAN bus devices are available

`~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`
you should see something like this:

```
Found canbus_uuid=a70885e01a2a, Application: Klipper
Found canbus_uuid=3b1b4f8763a2, Application: CanBoot
```

The Klipper device is Manta M8P just flashed
The CanBoot device is my SB 2209 CAN

If a davice says "Application: CanBoot" it means only the bootloader is saying hello and you have to program the klipper image in it.

So in my case SB 2209 needed klipper.img
so...

### 5. [Prepare klipper image for it and flash SB 2209 over can bus](doc/sb2209can.md)

---

### <u>Helpful videos</u>:

#### 1. [Bigtreetech 必趣 Manta M8P v1.1 + EBB Canbus CanBoot Klipper 完整教程](https://www.youtube.com/watch?v=ekbxtDS_8cM&t=327s)

by Botio 波提歐
It is in Chineese, very fast, but can be followed by looking at screens

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
   bitrate 250000
   up ifconfig $IFACE txqueuelen 1024
  ```

  **Note:** he uses 250000 in the video rather than 1000000, as I do

- query canbus
  `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`

  to get canbus_uuid

He then goes connects a Can toolhead
and again performs canbus_quary on can0 to find the other device's canbus_uuid
which he adds to the printer.cfg in [mcu ebb36] section

After this both canbus devices are visible in Klipper but no longer visible in query

### 2. [Manta M8P+EBB CANbus Setup](https://www.youtube.com/watch?v=EA-oBfenxAE)

by Stacking Layers
This is wone is at somewhat slower pace and in English
