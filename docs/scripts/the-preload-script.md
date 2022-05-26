#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/preload`. 
#### Purpose:
To generate the app-list for a given category, and prioritizing minimum execution-time.
#### Usage:
To generate the app-list of the "Installed" category for yad:
```bash
~/pi-apps/preload yad 'Installed'
```
To generate the app-list for xlunch for the main page (no category)
```bash
~/pi-apps/preload /home/pi/pi-apps/preload xlunch ''
```
#### How it works:
This script is designed for maximum speed. As a result, it uses many tricks to run faster.
1. Compile the `genapplist-yad` program. This is a C program designed to improve preloading times, but if it fails, a bash-based fallback is used.
    - It uses `gcc` to compile the `/etc/genapplist-yad.c`file to the `/etc/genapplist-yad` binary.
    - This program is tested; if it fails to produce 5 lines of output for one input app, the program is deleted to use the bash-based fallback.
2. Determine if it's **necessary** to generate the app-list.
    - If nothing has been modified in the `apps` folder, the `settings` folder, the `update-status` folder, the `etc` folder, the `api` script, the `preload` script, and the `categories` file, **skip** generating the app-list, **return the contents** of the previous app-list in the `data/preload` folder, and **exit**.
3. If it is necessary to generate the app-list, use the `app_categories` function to generate a list of all apps in their respective categories.
4. Limit the list of apps to the current category. (so if we're viewing the "Installed" category, hide all apps that are not in there.)
5. If the "Shuffle App list" setting is enabled, shuffle the list of apps and categories now.
    - Note: categories are always displayed above apps.
6. Remove apps that are not compatible with the operating system's CPU architecture.
7. Generate the list of visible categories.
8. Generate the list of visible apps.
    - If the GUI-mode is yad, use the `genapplist-yad` program if it exists.
    - Otherwise, generate the list of apps the slower bash-based way.
9. Save the resulting app-list to a file in the `data/preload` folder. (so that next time `preload` is run, it might not have to generate anything)
10. In the background, place all app-icons in the disk cache. This improves load-time.
11. Output the resulting app-list.
12. Run the `preload-daemon` script in the background to preload other categories. That way, when using Pi-Apps, by the time you click any category, it has just been preloaded and is ready-to-go.