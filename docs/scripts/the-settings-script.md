## The `settings` script
#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/settings`. 
#### Purpose:
To manage settings for Pi-Apps.
#### Usage:
To launch the graphical settings window:
```bash
~/pi-apps/settings
```
To check for missing setting-values and fill them with the default values, use this flag:
```bash
~/pi-apps/settings refresh
```
To revert all settings to their default values:
```bash
~/pi-apps/settings revert
```
#### How it works:
Each setting is stored in *two* places in the main pi-apps folder:
- The setting's possible values, default value, and explanation is stored in the `etc/setting-params` folder.
  - For example, the `etc/setting-params/App List Style` file contains:
    ```
    #Pi-Apps can display the apps as a compact list (yad), or as a group of larger icons. (xlunch)
    yad
    xlunch-dark
    xlunch-dark-3d
    xlunch-light-3d
    ```
  - All commented lines are the explanation, also known as the tooltip.
    - For this file, it is:
      ```
      #Pi-Apps can display the apps as a compact list (yad), or as a group of larger icons. (xlunch)
      ```
  - The first uncommented line is the default value for the setting.
    - For this file, it is:
      ```
      yad
      ```
  - Subsequent uncommented lines are additional possible values.
    - For this file, they are:
      ```
      xlunch-dark
      xlunch-dark-3d
      xlunch-light-3d
      ```
- The setting's current value is stored in the `data/settings` folder. This is a single file that contains a single-line value.
  - For example, if I set the app list style to "xlunch-dark-3d", the `data/settings/App List Style` file will contain "`xlunch-dark-3d`".

This dual-file/dual-folder approach is necessary to retain your choice, while allowing there to be future updates to settings, and/or their possible values. (Remember, the `data` directory is never updated but the `etc` folder is.)