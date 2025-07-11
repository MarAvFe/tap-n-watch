# Video Playback

Let's first test the video player. Specifically, using formats `.avi`, `.mp4` and `.mkv`.

`cvlc shows/LittleEinsteins/LittleEinsteins_S1E1.mp4`

> Note. This does not work through SSH. Remember to connect the display to Raspberry Pi (Lost 30 min of debug in this).

## ==== To be Continued

Many troubles presented when trying to run videos from the service origin. In general, nothing happened and the service logs claimed no dbus or smth was found

```
sudo systemctl status rfid-logger
sudo systemctl status scan-watcher
tail -f rfid-watcher-events.txt
```
