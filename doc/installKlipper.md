1.  Install klipper image on CB1
    The image:
    https://github.com/bigtreetech/CB1/releases

    I used this SD card flasher:
    https://etcher.balena.io/

2.  put the flashed SD card to the SoC SD slot

3.  login to the CB1 running Klipper
    ssh: biqu@your.local.IP password: biqu

    To see what distro is installed
    `cat /etc/os-release`

    Mine is:

    ```
    PRETTY_NAME="BTT-CB1 2.3.3 Bullseye"
    NAME="Debian GNU/Linux"
    VERSION_ID="11"
    VERSION="11 (bullseye)"
    ```

4.  Build klipper image for the Manta MCU
    according to M8P pdf manual
    `make menuconfig`
    and select:

        [*] Enable extra low-level configuration options
        - Micro-controller Architecture (STMicroelectronics STM32)
        - Processor model (STM32G0B1)
        - Bootloader offset (8KiB bootloader)
        - Clock Reference (8 MHz crystal)
        - Communication interface (USB (on PA11/PA12))

    then do:
    `make`
    and get the file: `home/pi/kliiper/out/klipper.bin` and

5.  Rename klipper.bin to firmware.bin
    put the file on the SD card
    reset Manta
    firmware will be updated and filename changes to FIRMWARE.cur

6.  List serial devices
    `ls /dev/serial/by-id`
    Mine shows:
    `usb-Klipper_stm32g0b1xx_3F004E0011504B4633373520-if00`
    For me the serial number 3F004E... didn't show before I installed klipper.bin in step 5

    This will be needed to connect with MCU over serial, e.g. in Klipper
    **printer.cfg**:

        [mcu]
        serial: /dev/serial/by-id/usb-Klipper_stm32g0b1xx_3F004E0011504B4633373520-if00

    Now Manta m8p works CB1 with MCU with the klipper installed

As of July 2023 BTT Manta m8p manual ends here
