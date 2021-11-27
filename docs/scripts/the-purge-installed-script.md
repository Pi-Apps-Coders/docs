### The `purge-installed` script
#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/purge-installed`. 
#### Purpose:
This script is the opposite of `pkg-install`: it uninstalls the dummy deb, then runs `sudo apt autoremove`.
#### Usage:
In an app's `uninstall` script, there may be a line like this:
```bash
DIRECTORY=$HOME/pi-apps
"${DIRECTORY}/purge-installed" "$(dirname "$0")" || exit 1
```