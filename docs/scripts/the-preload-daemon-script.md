## The `preload-daemon` script
#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/etc/preload-daemon`.  
This file is located in the `etc` folder because it's one of the smaller scripts.
#### Purpose:
To generate the app-list for all categories.
#### Usage:
Preload all categories for `yad`, repeating every 30 seconds for a total of 20 times:
```bash
~/pi-apps/etc/preload-daemon yad
```
Preload all categories for `xlunch` only once:
```bash
~/pi-apps/etc/preload-daemon xlunch once
```