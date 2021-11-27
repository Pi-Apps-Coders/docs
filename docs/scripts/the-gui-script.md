## The `gui` script
#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/gui`. 
#### Purpose:
This script handles Pi-Apps's entire user-interface.
#### Usage:
To run Pi-Apps: 
```bash
~/pi-apps/gui
```
To start Pi-Apps at a specific app:
```bash
~/pi-apps/gui Arduino
```
To start Pi-Apps on a specific category:
```bash
~/pi-apps/gui 'All Apps/'
```
#### How it works:
1. The splash screen appears.
2. The `api` script is sourced.
3. The pi-apps logo is displayed in the terminal (using the `generate_logo` function)
4. A series of `runonce` entries are executed in the background.
5. The message of the day is determined.
    - To save time, it's stored in `data/announcements`.
    - If that file is missing or it's more than a day old, it is downloaded from [the pi-apps-announcements repository](https://github.com/Botspot/pi-apps-announcements).
    - One random line is taken from the file and used as the message for this session.
6. We now come to a `while` loop that runs the GUI. Inside is an `if` statement that obeys the following values of the `$action` variable:
    - `main-window` - Handles the app list.
      - This may be a yad window or an xlunch window, depending on the "App List Style" setting.
      - Xlunch is compiled, if necessary.
    - `details` - Displays the details of the current app.
    - `search` - Sets the `$app` variable to the output of the `app_search_gui` function.
    - The rest of the modes need no explanation. They are: `exit`, `back`, `install`, `uninstall`, `scripts`, `edit`, `credits`, `enable`, `viewlog`, `mind-reading`, `view-updates`, `unknown`.