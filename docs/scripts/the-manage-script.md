#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/manage`. 
#### Purpose:
The `manage` script will install apps, uninstall apps, and update apps. It can be compared to the `apt` tool on Debian Linux.
#### Usage:
The manage script won't do much if you run it standalone:
```
$ ~/pi-apps/manage
You need to specify an operation, and in most cases, which app to operate on.
```
You need to tell the `manage` script to *do* something.
```
$ ~/pi-apps/manage install Arduino
```
Now we're getting somewhere! You just installed the Arduino app.
The `manage` script supports these **modes**:
- `install`: installs the specified app.
  Several things occur before the app's installation script is run:
  - The specified app must exist.
  - The app must not be disabled. If it is, the `manage` script exits with an exit-code of zero.
  - If your system is unsupported (determined by the `is_supported_system` function in the api script), a warning will appear, along with a 10-second wait-time.
  - The app's installation script is determined. Depending on the app and on your system's CPU architecture, the script-name may be "`install`", "`install-32`", or "`install-64`".
  - Determine a unique filename for the log-file to be generated. (This file will store the entire output of the installation process.)
  - Finally, the app's installation script is executed.
    - It is executed with the `nice` command, to lower the priority of the process so that the rest of the system remains responsive, even while compiling.
    - Its output is redirected to the log-file, *and* to stdout. (usually the terminal)
  - If the app's installation script **succeeded** (if it exited with a return code of `0`):
    - The the log-file is renamed to `install-success-$app`
    - The `manage` script exits with a return code of `0`.
  - However, if the app's installation script failed (any return code except `0`):
    - The the log-file will be renamed to `install-fail-$app`.
    - The `manage` script exits with a return code of `1`.
- `uninstall`: exactly like the `install` mode except that it uninstalls the specified app.
  - These two modes are so similar that they share the same code!
- `install-if-not-installed`: Installs the specified app, only if it has not already been installed.
  - This mode is especially useful for apps that **need another app to be installed first**.
  For example, the **`Wine (x86)`** app requires Box86. It accomplishes that with this comand:
  ```bash
  "${DIRECTORY}/manage" install-if-not-installed Box86 || error "Box86 failed to install somehow!"
  ```
- `multi-install`: installs multiple apps, one at a time. How to specify multiple apps? By using a ***multi-line argument***, like this:
  ```bash
  $ ~/pi-apps/manage multi-install "Arduino
  BalenaEtcher
  CloudBuddy
  Downgrade Chromium"
  ```
  Note about `multi-install`: This mode includes 
  elements.
  - Before installing anything, `manage` will check if any apps are **already installed**. If so, a `yad` dialog will appear and ask if you *really* want to install that app again.
      - If you choose "No", the app is removed from the list of apps to install.
  - Then, each app will be installed, one at a time.
      - It does this by running the manage script in the `install` mode, once for each app
  - If any apps fail to install, a `yad` dialog will appear and ask permission to send the error log to Pi-Apps developers.
- `multi-uninstall`: exactly like `multi-install` except that it uninstalls the list of apps.
  - These two modes are so similar that they share the same code!
- `check-all`: This mode is the Pi-Apps equivalent to an `apt update`. It lists updatable apps.
  - It downloads the latest pi-apps repository to the `update/pi-apps` folder. (using `git clone` or `git pull`, as appropriate) Now, there are two versions of Pi-Apps on the local filesystem: the "**local version**" and the "**latest version**".
  - Each app-folder is compared.
      - If the app-folder ***only*** exists in the local version, then no action is taken.
      - If the app-folder ***only*** exists in the online version, then it must be a new app and is added to the list of updatable apps.
      - If the app-folder exists in both locations and the **contents do match**, then no action is taken.
      - If the app-folder exists in both locations but the **contents don't match**, the online version must have received an update. As a result, the app is added to the list of updatable apps.
  - Finally, the list of updatable apps (one app per line) is written to standard output and the script exits.
- `update`: This mode will update a single app. (like an `apt upgrade`)
  It copies the new version from the update folder to the main folder, reinstalling if necessary.
  - First, the app may need to be installed, or it may not:
      - If the app is currently installed, *and* its current installation script does not match the online version, then the app is uninstalled.
  - Then the current (old) app-folder is sent to the system's **Trash** folder.
      - This is a failsafe: just in case you made changes to the app-folder, you have an option to restore those changes. (as opposed to permanent deletion)
  - The app-folder is copied from the `update/pi-apps/apps` folder to the main `apps` folder.
  - If the app was uninstalled earlier, it will now be installed back.
- `update-all`: This mode will check for app-updates and install them without any user-interaction.
  The `manage` script will run itself in the `check-all` mode, then, for every app that `check-all` mentioned, it will `update` each app.