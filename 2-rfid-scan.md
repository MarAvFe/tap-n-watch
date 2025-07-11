# RFID Scanner Testing

First we need to see the events happening and being read by the scanner, with reproducible values for each tag.

The Raspberry Pi Zero has only 2 ports. So we need to connect into the pi through SSH, to connect the RFID Scanner on the data usb port.

The scanner works as a keyboard, where each scan is a keystroke.

## 1. Does the scanner work?

1. Identify the scanner with: `ls /dev/input/by-id/usb-IC_Reader_IC_Reader_08FF20171101-event-kbd`. The last part is the scanner id.
2. Run `cat /dev/input/by-id/SCANNERID` and scan some tags, to see "something happening". The scanner works but this is trash data.

```sh
marce@raspberrypi:~ $ cat /dev/input/by-id/usb-IC_Reader_IC_Reader_08FF20171101-event-kbd
�Xph���Xph���Xph���Xphu��Xphu��Xphu��Xph:$�Xph�Xph:�Xphnx$�Xphn�Xphnx�Xph��$�Xph��Xph���Xphs�$�Xphs�Xphs��Xph�3$�Xph��Xph�3�Xphar$�Xpha�Xphar�Xph��Xph��Xph��Xphq��Xphq��Xphq��Xph�-'�Xph�-
                                        �Xph�-�Xph^l'�Xph^l
                                                           �Xph^l�Xphڪ�Xphڪ�Xphڪ�Xph[��Xph[��Xph[��Xph��Xph��Xph��XphE�XphE�XphE�Xph"�Xph�Xph�Xph"�XphU�XphU�Xph�! &�Xph�!
�Xph�!  �Xph>`  &�Xph>`
�Xph>`  �Xphƞ   (�Xphƞ  �Xphƞ   �XphP�  (�XphP� �XphP�
```

## 2. Identify One Tag from the Others

1. Create this Python file `vim log_rfid_evdev.py` (verify the `DEVICE_PATH` and `LOG_FILE` variables)

```py
#!/usr/bin/env python3

from evdev import InputDevice, categorize, ecodes
from datetime import datetime

DEVICE_PATH = "/dev/input/by-id/usb-IC_Reader_IC_Reader_08FF20171101-event-kbd"
LOG_FILE = "/home/marce/rfid_scans.log"

dev = InputDevice(DEVICE_PATH)
buffer = ""

print(f"{datetime.now().isoformat()} Listening...\n")
for event in dev.read_loop():
    if event.type == ecodes.EV_KEY:
        key = categorize(event)
        if key.keystate == key.key_down:
            if key.keycode == 'KEY_ENTER':
                with open(LOG_FILE, "a") as f:
                    f.write(f"{datetime.now().isoformat()} {buffer}\n")
                buffer = ""
            else:
                buffer += key.keycode.replace('KEY_', '')
```

2. Make the file executable `chmod +x log_rfid_evdev.py`
3. Run it `./log_rfid_evdev.py` and open a new shell tab.
4. Create the output file with `touch /home/marce/rfid_scans.log`
5. Run `tail -f /home/marce/rfid_scans.log` and scan some tags

```
 $ tail -f rfid_scans.log
2025-07-11T01:15:05.734163 2777102259
2025-07-11T01:15:07.916756 2777102259
2025-07-11T01:15:11.237903 2788102755
2025-07-11T01:15:13.597815 2777102259
2025-07-11T01:15:15.940658 2777102259
2025-07-11T01:15:18.038298 2788102755
2025-07-11T01:20:58.637869 2777102259
```

## 3. Making the Scanning Script Run as a Service

1. Create the systemd service file: `sudo vim.tiny /etc/systemd/system/rfid-logger.service`. Check on the `user` path and variable.

```ini
[Unit]
Description=RFID Scanner Logger
After=multi-user.target

[Service]
ExecStart=/home/marce/log_rfid_evdev.py
Restart=on-failure
User=marce

[Install]
WantedBy=multi-user.target

```

2. Enable the service

```sh
sudo systemctl daemon-reload
sudo systemctl enable rfid-logger.service
sudo systemctl start rfid-logger.service
```
3. Monitor the output file again

```
$ tail -f rfid_scans.log
2025-07-11T01:31:31.775323 2788102755
2025-07-11T01:31:33.748652 2788102755
```
