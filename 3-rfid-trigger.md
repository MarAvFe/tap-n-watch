# Trigger an Event on RFID Scan

The test service is great at what it does. Leave it simple.

Let's watch it's output and identify the tags to trigger events on each one. This was it's output:

```
$ tail -f rfid_scans.log
2025-07-11T01:31:31.775323 2788102755
2025-07-11T01:31:33.748652 2788102755
```

## 1. Watch the Scans

This Python script will also run as a service, listning for new lines in the scanner's output

1. Create it with `vim.tiny scan-watcher.py`

```py
#!/usr/bin/env python3

import time
import subprocess

LOG_FILE = "/home/marce/rfid_scans.log"

# === Simple tag-to-show mapping database ===
TAG_DATABASE = {
    "2703104131": "Backyardigans",
    "2777102259": "Zoboomafoo",
    "2788102755": "Little Einsteins",
    # Add more tag IDs and their shows here
}

def handle_scan(tag_id):
    show = TAG_DATABASE.get(tag_id)
    if not show:
        print(f"[WARN] Unknown tag: {tag_id}")
        return

    print(f"[INFO] Playing: {show}")
    # Placeholder: print the show name. Replace this line with actual video playback
    subprocess.run(["echo", f"Playing {show}"])

def watch_log():
    last_line = ""
    with open(LOG_FILE, "r") as f:
        f.seek(0, 2)  # Move to end of file
        while True:
            line = f.readline()
            if line and line != last_line:
                last_line = line
                tag_id = line.strip().split()[-1]
                handle_scan(tag_id)
            time.sleep(0.1)

if __name__ == "__main__":
    watch_log()
```

2. Make it executable with `chmod +x scan-watcher.py`
3. Run it and check the output `./scan-watcher.py`

```sh
$ ./scan-watcher.py
[INFO] Playing: Backyardigans
Playing Backyardigans
[INFO] Playing: Zoboomafoo
Playing Zoboomafoo
[INFO] Playing: Backyardigans
Playing Backyardigans
```

## 2. Make It a Service

1. Create the service file: `sudo vim.tiny /etc/systemd/system/scan-watcher.service`

```ini
[Unit]
Description=RFID Scans Watcher
After=multi-user.target

[Service]
ExecStart=/home/marce/scan-watcher.py
Restart=on-failure
User=marce

[Install]
WantedBy=multi-user.target
```

2. Enable the service

```sh
sudo systemctl daemon-reload
sudo systemctl enable scan-watcher.service
sudo systemctl start scan-watcher.service
```

3. Scan a tag and run `sudo systemctl status scan-watcher.service` to see logs:

```ini
$ sudo systemctl status scan-watcher.service
● scan-watcher.service - RFID Scans Watcher
     Loaded: loaded (/etc/systemd/system/scan-watcher.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-07-11 02:00:45 BST; 1min 50s ago
   Main PID: 21160 (python3)
      Tasks: 1 (limit: 386)
        CPU: 891ms
     CGroup: /system.slice/scan-watcher.service
             └─21160 python3 /home/marce/scan-watcher.py

Jul 11 02:00:45 raspberrypi systemd[1]: Started scan-watcher.service - RFID Scans Watcher.
Jul 11 02:01:12 raspberrypi scan-watcher.py[21172]: Playing Backyardigans
Jul 11 02:02:33 raspberrypi scan-watcher.py[21190]: Playing Little Einsteins
```
