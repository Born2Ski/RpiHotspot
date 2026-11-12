# RpiHotspot

Adds the Wi-Fi access point of the GX devices to Venus OS on a Raspberry Pi,
using the Pi's built-in wifi. Like on a GX device it is switched on / off and
given a password in the GUI (Settings / Wi-Fi), clients get an address in
172.24.24.0/24 and the device is reachable at 172.24.24.1 / venus.local.

Requires [SetupHelper](https://github.com/kwindrem/SetupHelper), which also
reinstalls the package after a Venus OS firmware update.

## How it works

venus-platform and the GUI already support the access point. Once the hostapd
service template exists they show the settings and add / remove
`/service/hostapd`. Venus OS just doesn't ship hostapd for the Raspberry Pi, so
the setup script installs:

- `/opt/victronenergy/service-templates/conf/hostapd.conf`
- `/opt/victronenergy/service-templates/hostapd/` (`run` starts `hostapd-run`
  from this package)

and the `Settings/Services/AccessPointPassword` setting, and restarts
venus-platform.

`hostapd-run` adds the virtual interface `ap0` to the built-in wifi (brcmfmac),
runs dnsmasq for DHCP on it and starts hostapd (from the Venus OS v3.85
`cortexa7hf-neon-vfpv4` feed, see `bin/`).

## Install

Copy this directory to `/data/RpiHotspot` on the Pi and run
`/data/RpiHotspot/setup`, or add the package in PackageManager.

Then set an Access Point password (8 to 63 characters) in Settings / Wi-Fi. The
Pi has no factory wifi password, so the access point doesn't start before that.
The network is called `venus-<unique id>`.

Logs: `/var/log/hostapd/current`

## Limitations

- Raspberry Pi 2 / 3 / Zero 2 W (`raspberrypi2`) and Raspberry Pi 4
  (`raspberrypi4`) images only. The Pi 5 image is 64 bit and would need another
  hostapd binary.
- Venus OS v3.85 or later (the bundled hostapd is linked against its libraries).
- The built-in wifi can only use one channel at a time. When the Pi is also
  connected to a wifi network, the access point uses the channel of that
  network and is restarted when that channel changes. With ethernet only, it
  uses channel 1.
- Like on a GX device, the access point gives local access only, no internet.
