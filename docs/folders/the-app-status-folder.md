Pi-Apps keeps track of each app's status: installed or uninstalled. (as well as a few other possible values)  
Each app's status is located in the `data/status` folder.  
For example, on a default Pi-Apps installation, the status file for **arduino** is located at: `/home/pi/pi-apps/data/status/Arduino`.  
Each file in this folder will contain one of these possible values:
- `installed` - The app is currently installed.
- `uninstalled` - The app is currently uninstalled.
- `corrupted` - The app failed to install/uninstall.
- `disabled` - The app is in a disabled state: it will not be installed under any circumstances. 
  - This status is useful for TwisterOS where Box86 comes pre-installed. We don't want Pi-Apps installing Box86 under any circumstances, even if another app requires it to be installed.

As mentioned earlier, Pi-Apps runs on bash scripts. Each script has a certain job to do, and together they make Pi-Apps work.  

See the scripts folder for how each individual script works.