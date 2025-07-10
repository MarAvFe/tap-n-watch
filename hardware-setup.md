# Hardware setup

Some steps that don't come out of the box from tutorials

## List of items

1. Raspberry Pi Zero W
2. SDHC Card
3. Mini HDMI to HDMI adapter
4. HDMI Cable
5. MicroUSB to USB OTG (keyboard, RFID reader)
6. Power MicroUSB cable

## Raspberry Pi Headless Config

### 1. Flashing the OS

- Used [**Raspberry Pi Imager**](https://www.raspberrypi.com/software/) to flash **Raspberry Pi OS Lite (32-bit)** to the microSD card.
- In Imager’s advanced options (⚙️):
  - Enable **hostname**
  - Set **username and password**
  - Set Timezone to `America/Costa_Rica` and keyboard to `us`. (Else defaults to `uk`)
  - Enable **SSH** (either via Imager or manually)

### 2. First Boot (with direct hdmi display control)

- Inserted microSD and booted Pi with HDMI and keyboard.
- Logged in.
- Confirmed `wlan0` was not connected. (`ip a show` has no IP)

### 3. Disabled RF Kill WiFi block

On login prompt, in a repeatable way I got 

```
Wi-Fi is currently blocked by rfkill
Use raspi-config to set the country before use.
```

So to validate and workaround it (didn't see the raspi-config instruction on time)

```
rfkill list
sudo rfkill unblock wifi
```

### 4. Create Wi-Fi Config

- WiFi config credentials file: `sudo vim.tiny /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`

```
country=CR  
ctrl_interface=/var/run/wpa_supplicant GROUP=netdev
update_config=1  

network={  
  ssid="MySSID"  
  psk="MyPassword"
}
```

- DHCP Config on Boot: `sudo vim.tiny /etc/systemd/network/wlan0.network`

```
[Match]
Name=wlan0

[Network]
DHCP=yes
```

⚠️ The filename **must match the interface** (`wlan0`) (`ip a show`).

### 5. Enabled Auto-Start of Wi-Fi and DHCP

- Enabled the systemd service for interface `wlan0`:

```
sudo systemctl enable wpa_supplicant@wlan0
sudo systemctl enable systemd-networkd
```

- Rebooted and verified:

```
sudo reboot
systemctl status wpa_supplicant@wlan0  
ip a show wlan0  
iw wlan0 link
```

- Confirmed connection and IP assignment.

### 6. Confirmed DHCP

- DHCP client `systemd-networkd` worked automatically.
- No need to manually request an IP after boot.

### 7. Final State

- Pi boots headless.
- Connects to Wi-Fi automatically using `wpa_supplicant@wlan0`.
- Gets IP without manual intervention.
- Fully accessible via SSH.

### Troubleshooting Tips

- To check if Wi-Fi config is loaded:

```
journalctl -u wpa_supplicant@wlan0  
dmesg | grep wlan
```
- If no IP:

```
ip a  
ping 8.8.8.8
```

- If not connecting:
  - Check that the config file matches the interface name.
  - Ensure `rfkill` is not re-blocking the interface after reboot.
