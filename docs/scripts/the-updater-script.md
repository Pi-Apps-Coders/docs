---
template: overrides/main.html
---

#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/updater`. 
#### Purpose:
This script handles Pi-Apps updates, both for apps and for files. Apps do not have version numbers, so updating involves comparing the installation-script of the existing app-folder with the installation-script of the new app-folder.
#### Usage:
```bash
~/pi-apps/updater gui
```
This will check for updatable apps and files, then will display them in a graphical list, (using `yad` of course), and will give you the option to update certain items and not others.  
But that's not all you can do with this script. The `updater` script supports multiple **modes**, useful for a variety of situations:
- `gui` - Check for updates and list them graphically.
  - This is the **default mode** if you did not specify one.
  - If you choose to update anything, a **terminal** will appear to update apps.
- `gui-yes` - Exactly the same as the `gui` mode, except that the updates are automatically applied in a terminal.
- `autostarted` or `onboot` - This mode is intended to be **run on boot**.
  - First, determine if the **update-interval** setting allows an update-check today.
    - If not, the script exits.
  - Then, make sure that **at least one app has been installed**.
    - This is accomplished by checking if there are any files in the `data/status` folder.
    A fresh installation of Pi-Apps will not have any files in that folder, because no apps have been installed yet.
    - If there are no files in that folder, the script will exit.
  - After that, assuming there are updates available, display a **notification** in the lower-right corner of the screen.
    - This notification is designed to not interfere with **typing**.
  - If you click **Details**, you will see the graphical list of updates, just like with the `gui` mode.
  - Once updates are complete, another notification will appear to say "Pi-Apps updates complete."
- `cli` - This mode is intended to be run in a **terminal**. It checks for updates, lists the updates, and prompts for a Y/N answer.
- `cli-yes` - Check for updates, then **automatically apply** them all.
- `set-status` - **Check for updates** and then exit.
  - The list of updatable **apps** is written to `"${DIRECTORY}/data/update-status/updatable-apps"`, with the DIRECTORY variable being the location of your Pi-Apps folder. (default: `/home/pi/pi-apps`)
  - The list of updatable **files** is written to `"${DIRECTORY}/data/update-status/updatable-files"`.
  - The script will exit with code `0` if updates are available, otherwise `1`.
- `get-status` - Check if an **update *was* available** the last time `set-status` was run.
  - This is based on the **length** of the `updatable-files` and `updatable-apps` files.
  - Doing it this way allows for an **instant** update-check - necessary for a gui to run quickly. Nobody wants to wait for an update-check to finish before Pi-Apps will launch.
  - The script will exit with code `0` if updates are available, otherwise `1`.

Updater also supports a **fast mode**. This is necessary for the Pi-Apps GUI, where you don't want to wait a few seconds after clicking the "Updates" category. Using the fast mode will rely on the previous update-check. It won't check for updates, it will simply display them.  
To use the fast mode:
```bash
~/pi-apps/updater gui fast
```