# Trolley OS testing and rollout notes

> PB Steen
> June 2021
> Ubuntu 20.04 LTS

## Particle cli setup (linux)

**use shell install script, not npm!** (javascript always eventually breaks... see dfu- & serial-utils issues)

From [particle docs](https://docs.particle.io/tutorials/developer-tools/cli/#installing):

```shell
bash <( curl -sL https://particle.io/install-cli )
```

No real need for the particle-workbench vscode tool, would rather avoid if I can due to increased process opacity.
To *really* learn how the particle/photon stack works, I should try to live in my terminal!


### Interfacing issues

If `dfu-utils` is still a pain, have another look at what is going on with the `50-particle.rules` file.
Is there a reason that the [fix](https://support.particle.io/hc/en-us/articles/360039251394/) suggested by particle doesn't seem to fix anything?

Running

```shell
sudo apt-get install dfu-util
```

is suppposed to fix all my problems along with the `50-particle.rules` *'hack'*.
We shall see...


## Manual photon os and system flashing procedure

**In the event that the dev tools install is successful...**

The procedure for setting up a photon as it would be at the factory setup stage is as follows:

1. Acquire the appropriate device-os binaries from [particle's github](https://github.com/particle-iot/device-os/releases), we need:
	- photon-1.4.4
	- photon-2.1.0
2. Make sure you get the following binaries for each firmware version:
	1. `photon-bootloader@1.4.4+lto.bin*`
	2. `photon-system-part1@1.4.4.bin*`
	3. `photon-system-part2@1.4.4.bin*`
	4. `photon-tinker@1.4.4.bin*`
3. Flash the device-os/firmware to the photon over usb:
	1. Put photon into DFU mode:
		1. Press and hold both the `RESET/RST` and `MODE/SETUP` buttons simultaneously
		2. Release only the `RESET/RST` button while continuing to hold the `MODE/SETUP` button
		3. Release the `MODE/SETUP` button once the device begins to blink yellow
	2. Check that the photon is visible to the flashing device:
		```shell
		particle list
		```
	3. If the device is in **dfu-mode** and visible to the system, use the `particle-cli` tool to flash the device-os to the duf-mode photon with:
		```shell
		particle flash --usb <firmware>.bin
		```
		**The order in which you flash the os binaries is very important!** Wait for flash success/fail message before proceeding to next flash instruction.
		```shell
		particle flash --usb photon-system-part1@<version>.bin
		particle flash --usb photon-system-part2@<version>.bin
		particle flash --usb photon-tinker@<version>.bin
		```
	4. Next, before flashing the bootloader to the photon, we need to the put the device into **serial-mode**. We do this by pressing `RESET/RST` once until the LED is flashing white, then hold `MODE/SETUP` until it flashes blue. The device is now in serial mode and we can flash the bootloader to it using:
		```shell
		particle flash --serial photon-bootloader@<version>+lto.bin
		```
