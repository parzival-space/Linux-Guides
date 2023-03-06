# Setting up OpenRGB
This guide will cover the process of setting up OpenRGB, the ultimate tool for controlling your RGB components and creating the perfect lighting setup for your system.

Before you get started with OpenRGB, make sure your hardware is compatible.
You can check out the list of supported devices on the official OpenRGB website at <a href="https://openrgb.org">https://openrgb.org</a>.

## Installing OpenRGB
Installing OpenRGB is easy. 
Simply use your favorite AUR wrapper to run one of the following commands:
* Stable Release:
  ```bash
  pamac install openrgb
  ```
* Latest Build (unstable):
  ```bash
  pamac install openrgb-git
  ```

## Enabling SMBus Access
To control RGB RAM and certain motherboard on-board LEDs with OpenRGB, you'll need to enable SMBus access. If you're not planning to use OpenRGB for this purpose, you can skip this section.

### ASUS and ASRock Motherboards
Note that ASUS and ASRock motherboards have their RGB controller on a secondary SMBus interface and require a Linux kernel version higher than 5.7.

### Load Modules
To enable SMBus access, you'll need o load the SMBus module, run the following command in the terminal:
```bash
sudo modprobe i2c-dev
```

You also need to enable the according chipset driver for your system.
* Intel:
  ```bash
  sudo modprobe i2c-i801
  echo "i2c-i801" | sudo tee /etc/modules-load.d/openrgb.conf
  ```

* AMD:
  ```bash
  sudo modprobe i2c-piix4
  echo "i2c-piix4" | sudo tee /etc/modules-load.d/openrgb.conf
  ```

## Trigger UDEV update
You'll need to trigger a udev update so OpenRGB can detect your USB devices in the current session. This can be done in two ways:
* Re-plug all your USB devices
* Run the following command to trigger a udev update:
  ```bash
  sudo udevadm control --reload-rules && sudo udevadm trigger 
  ```

## Thats it
Congratulations, you have successfully installed OpenRGB and enabled SMBus access.
Now you can easily create the perfect lighting setup for your system.
Don't forget to check out the official documentation for more information on how to use the more advanced features of OpenRGB.
Enjoy customizing your RGB components and creating an immersive visual experience for your system!

## Final Notes
Here are some important things to keep in mind when using OpenRGB:

* You can run OpenRGB with a pre-configured profile on startup by following one of the 2 documented methods available in the <a href="https://gitlab.com/CalcProgrammer1/OpenRGB/-/wikis/Frequently-Asked-Questions#can-i-run-openrgb-at-startup-on-linux">official documentation</a>.

* OpenRGB cannot detect ovmf devices because they no longer use their original drivers and instead use the ovmf driver.