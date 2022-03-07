---
template: overrides/main.html
---

#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/etc/viewlog`.  
This file is located in the `etc` folder because it's one of the smaller scripts.
#### Purpose:
View the specified log file in a text editor.
#### Usage:
```bash
~/pi-apps/etc/viewlog ~/pi-apps/logs/install-success-Arduino.log
```
Notes:
- This script will kill previous instances of itself.
- This script **does not obey** the "Preferred text editor" setting.
  - Why? Viewing a log in ***`geany`*** (the default "Preferred text editor" setting) is cumbersome and breaks the whole concept of closing the editor once you click on a new logfile.