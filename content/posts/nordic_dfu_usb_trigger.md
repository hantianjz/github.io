---
title: Trigger DFU via USB on nRF52840 Dongle
date: 2024-04-06
img:
---




The [NRF5280 dongle](https://www.nordicsemi.com/Software-and-Tools/Development-Kits/nRF52840-Dongle) is a great embedded development platform, especially for hobbyist project or quick prototyping. There is also a very similar product from [Make Diary](https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/programming/) with slightly different PCBA layout. It is generally a decent utility tool for BLE related work. These functionality can all be done out of box with the nrf desktop tool without doing any programming.

There are plenty of blog post on this already that covers introductory programming with the nordic part.
But one interesting feature that I run into while working with this dongle was how to trigger DFU programming quickly.

The nRF52840 dongle support DFU programming via it's bootloader, where one can use `nrfutil` program the hardware.
The dongle can be reboot into DFU mode via the side reset button.
![NRF52840 dongle pcb black white](/img/NRF52840_dongle_pcb_black_white.png)
The dongle's red LED goes into this slow glow effect, and the usb device is re-enumerated as 
`ID 1915:521f Nordic Semiconductor ASA Open DFU Bootloader`
All this mean the dongle is ready for DFU programming now!

Now you can a command like to issue the programming:
```bash
$ nrfutil device program --firmware dfu_package.zip --traits nordicDfu 
```

In addition you can create the `dfu_package.zip` file with:
```bash
$ nrfutil nrf5sdk-tools pkg generate --hw-version 52 --sd-req=0x00 --application application.zip --application-version 1 dfu_package.zip
```

This flow works great for 90% of development flows. You will need to be careful not to accidentally erase the UICR region, or else the bootloader will no longer boot into DFU. I learned this the hardware. If you do end up get the hardware into a state where bootloader don't enter DFU mode, you will need to get a jtag programmer and connect the jtag pins to the dongle to programming the old fashion way.

But these are not what I want to focus on here, I want to focus on how to trigger the DFU mode to begin with.

I am a lazy person, the idea of having to manually press a button on the hardware to start FW programming is just very tiresome to me. Not to mention doing this tens or hundreds times a day.

# Triggering DFU mode via USB
Essentially on the nRF52840 dongle, just setting the reset pins cause the MCU to reboot into bootloader and bootloader code detect that reset pin are set and enter into DFU mode.
```c
nrf_gpio_cfg_output(BSP_SELF_PINRESET_PIN);
nrf_gpio_pin_clear(BSP_SELF_PINRESET_PIN);
```
The hard part is somehow getting a signal from the host machine to the target device to reset the pin.

But luckily this is all reasonably [documented](https://infocenter.nordicsemi.com/topic/sdk_nrf5_v17.1.0/lib_dfu_trigger_usb.html)!
In addition there is a official [standard](https://www.usb.org/sites/default/files/DFU_1.1.pdf) for this, Nordic implementation isn't fully compliant but it's close enough.
The nrfsdk provide this helpful [component](https://infocenter.nordicsemi.com/topic/sdk_nrf5_v17.1.0/group__nrf__dfu__trigger__usb.html) anyone can simply include in their FW, some configuration with `sdk_config.h` is required, as with everything in nrfsdk.

If setup correct it should show up as new vendor interface similar to.
```bash
DEVICE ID 05ac:0256 on Bus 000 Address 007 =================
...
  CONFIGURATION 1: 100 mA ==================================
  ...
    INTERFACE 0: Vendor Specific ===========================
     bLength            :    0x9 (9 bytes)
     bDescriptorType    :    0x4 Interface
     bInterfaceNumber   :    0x0
     bAlternateSetting  :    0x0
     bNumEndpoints      :    0x0
     bInterfaceClass    :   0xff Vendor Specific
     bInterfaceSubClass :    0x1
     bInterfaceProtocol :    0x1
     iInterface         :    0x0
```

Therefore by setting up a control transfer to the usb device like below, it should in theory trigger the dongle to reboot in DFU mode.
```python
libusb1_backend = usb.backend.libusb1.get_backend(
    find_library=lambda x: "/opt/homebrew/lib/libusb-1.0.0.dylib"
)

dev = usb.core.find(idVendor=0x1915, idProduct=0x521f, backend=libusb1_backend)

bmRequestType = usb.util.build_request_type(
    usb.util.CTRL_OUT, usb.util.CTRL_TYPE_CLASS, usb.util.CTRL_RECIPIENT_DEVICE
)

try:
    dev.ctrl_transfer(bmRequestType, 0x00, timeout=1)
except usb.core.USBError:
    pass
```

If you noticed the dongle rebooting but not entering DFU, it's likely resetting or crashing somewhere before the reset pin is set. Just check the code path to `dfu_trigger_evt_handler()` in `components/libraries/bootloader/dfu/nrf_dfu_trigger_usb.c` function free of other issue. For example I had used the log backend, but didn't set it up correctly and it was causing the dongle to crash on `NRF_LOG_FINAL_FLUSH()` right before the reset pin is set.
