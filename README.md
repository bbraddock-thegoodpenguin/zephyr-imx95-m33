# Zephyr for the i.MX95 M33

I have the following forks
- [zephyr](https://github.com/bbraddock-thegoodpenguin/zephyr)
- [hal_nxp](https://github.com/bbraddock-thegoodpenguin/hal_nxp)

This is currently a very rough proof of concept only, none of this is intended to be upstreamed in this state.
The intent here was to prove that it was possible for Zephyr to run on the i.MX95 M33 core using the existing support for the i.MX93 M33.


# Building
I have tested the hello world so far
```Bash
west build -b imx95_evk/mimx9596/m33 samples/hello_world
```

```
*** Booting Zephyr OS build v4.4.0-4780-g00ce7750e283 ***
Hello World! imx95_evk/mimx9596/m33
```

## Running & Debugging

The JTAG on the i.MX95 is pinmuxed with UART and is usually used for communicating with bluetooth on wireless modules; this means JTAG will only work if the hardware design allows for it

I am using the [FRDM i.MX95 PRO](https://www.nxp.com/design/design-center/development-boards-and-designs/FRDM-IMX95-PRO) which has JTAG support but is quite difficult to use because of the design.

[FRDM i.MX95](https://www.nxp.com/design/design-center/development-boards-and-designs/FRDM-IMX95) will run the firmware but does not support JTAG because of a design limitation

[i.MX95 EVK](https://www.nxp.com/design/design-center/development-boards-and-designs/IMX95LPD5EVK-19) has a standard JTAG header and DIP switches to isolate the JTAG interface from the wireless module

### I.MX95 FRDM PRO JTAG

(I would show pictures but as you need an account to access the design files, I'm not sure if I can)

You can download the [design files](https://www.nxp.com/webapp/Download?colCode=FRDM-IMX95-PRO-DESIGNFILES) and view `LAY-95794_B1.brd` with [Altium viewer](https://www.altium.com/viewer)

The board has test pads for JTAG but they are difficult to access as they are close together and inbetween the heatsink and 10G connection.
The easiest way I found was to use the resistors on the underside of the board

```
       ┌────┐  ┌────┐    ┌───────────┐
[TCK]  │R411├──┤R412├────┤           │
       └────┘  └────┘    │           │
       ┌────┐  ┌────┐    │           │
[TMS]  │R420├──┤R421├────┤           │
       └────┘  └────┘    │    U57    │
       ┌────┐  ┌────┐    │           │
[TDO]  │R428┼──┤R429├────┤           │
       └────┘  └────┘    │           │
       ┌────┐  ┌────┐    │           │
[TDI]  │R434┼──┤R435├────┤           │
       └────┘  └────┘    └───────────┘
```

You want to connect to the middle of these resistors (either side). I had two probes connected to R421 and R429, and two coming from the side to connect to R411 and R434

The other issue is that the 3v3 pin available are only active once the PMIC has enabled them, at the early stage, they are still 0v.
You need `VDD_3V3` which you can access via some unpopulated resistors (also on the underside of the board, just above the two type C connectors)
For resets I just used the reset button since it was accessible, you could use `C577` if you wanted as this is also near `VDD_3V3`

### Running Zephyr

With a working JTAG and the device in USB download mode you can load the firmware via `uuu` and `imx-mkimage` using `make SOC=iMX95 OEI=no flash_lpboot_sm_no_ahabfw` or use `gdb` directly

```Bash
JLinkGDBServer -if jtag -speed auto -device MIMX9596_M33
```

```Bash
gdb -ex "file zephyr.elf" -ex "target remote :2331" -ex "load"
```
