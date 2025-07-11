# Bootstrap Behavior when Identifying a Show

A few decisions were made at this point:

1. Split logic on 2 unix services to isolate the RFID scan events.
2. Use a "manual" database. The array of shows will never be huge, so it will be maintainable. And if it doesn't the project will get an update.
3. The kid will choose the show, but not the chapter. Hence those will be in folders and randomly selected.

```
/home/marce/shows/
├── Show1/
│   ├── lion_episode.mp4
│   └── jungle_fun.mkv
├── Show2/
│   └── rockets_and_colors.mp4
└── Show3/
    └── shapes_and_sounds.avi
```

4. The watcher will have a 10 second cooldown on new inputs to avoid multiple scans triggering multiple shows due to human error.

## Up the Game of the Script

1. Create add randomizer, logger and cooldown behavior

```py
#!/usr/bin/env python3

import time
import subprocess
import os
import random

LOG_FILE = "/home/marce/rfid_scans.log"
EVENT_LOG = "/home/marce/rfid_watch_events.log"
SHOWS_BASE_PATH = "/home/marce/shows"

TAG_DATABASE = {
    "2703104131": "Backyardigans",
    "2777102259": "Zoboomafoo",
    "2788102755": "LittleEinsteins",
    # Add more tag IDs and their shows here
}

COOLDOWN_SECONDS = 10
last_action_time = 0

def log_event(message):
    timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
    with open(EVENT_LOG, "a") as f:
        f.write(f"[{timestamp}] {message}\n")

def handle_scan(tag_id):
    global last_action_time
    now = time.time()

    if now - last_action_time < COOLDOWN_SECONDS:
        log_event(f"Scan ignored (cooldown): {tag_id}")
        return

    last_action_time = now
    show = TAG_DATABASE.get(tag_id)
    if not show:
        log_event(f"Unknown tag: {tag_id}")
        return

    show_path = os.path.join(SHOWS_BASE_PATH, show)
    if not os.path.isdir(show_path):
        log_event(f"Show folder not found: {show_path}")
        return

    files = [f for f in os.listdir(show_path) if f.endswith(('.mp4', '.mkv', '.avi'))]
    if not files:
        log_event(f"No videos found in {show_path}")
        return

    selected_file = random.choice(files)
    full_path = os.path.join(show_path, selected_file)
    log_event(f"Playing: {full_path}")

    # Later: Replace with actual player
    # subprocess.run(["omxplayer", full_path])

def watch_log():
    last_line = ""
    with open(LOG_FILE, "r") as f:
        f.seek(0, 2)
        while True:
            line = f.readline()
            if not line:
                time.sleep(0.1)
                continue
            if line != last_line:
                last_line = line
                tag_id = line.strip().split()[-1]
                handle_scan(tag_id)

if __name__ == "__main__":
    watch_log()
```

## Testing

1. Create the folders: `mkdir -p shows/{LittleEinsteins,Backyardigans,Zoboomafoo}`
2. Create the "videos": `touch shows/{LittleEinsteins,Backyardigans,Zoboomafoo}/chap{1,2,3,4}.avi`
3. Monitor the log: `tail -f rfid_watch_events.log`

```
[2025-07-11 02:23:14] Playing: /home/marce/shows/Backyardigans/chap1.avi
[2025-07-11 02:23:17] Scan ignored (cooldown): 2777102259
[2025-07-11 02:23:25] Playing: /home/marce/shows/Zoboomafoo/chap3.avi
[2025-07-11 02:23:38] Playing: /home/marce/shows/Zoboomafoo/chap4.avi
```
