#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/api`. 
<br>
#### Purpose:
This script is a collection of **functions** that do various things. Functions are small chunks of bash-code that can be run like a normal command.
<br>
#### Usage:
```bash
source ~/pi-apps/api
```
You can now run any of the functions inside the api script as if they were real commands.  
Alternatively, the `api` script supports running a single function *without* being sourced:
```bash
~/pi-apps/api apt_lock_wait
```
<br>
#### List of functions:
Note: new functions are added often. If you don't see a function on this list but do see it in the api, please let us know.
- `error` - display a custom message in red and exit with a return code of `1`.
  Usage:
  ```
  error "The command 'sudo apt update' failed!"
  ```
  This is often seen at the end of a command with the `||` operator:
  ```
  sudo apt update || error "The command 'sudo apt update' failed!"
  ```
- `warning` - Display a custom message in yellow and prefix it with "WARNING: ".
  - Useful for everything where something is wrong but it's not a fatal error.
  - This function outputs to `stderr`.
- `status` - Display a custom message in light-blue.
  - Used by scripts to indicate current status, like "Downloading...", "Extracting...", and "Please wait."
  - This function outputs to `stderr`.
  - Some scripts don't want the ending newline, so this function allows for flags to be passed to the `echo` command. Example usage: `status -n "Downloading... "`
- `status-green` - Display a custom message in green.
  - Used by scripts to indicate the success of an action, like "Installed FreeCAD successfully", "Update complete", and "All packages have been purged successfully."
  - This function outputs to `stderr`.
- `generate_logo` - Displays the Pi-Apps logo in a terminal.
- `add_english` - Ensures an English locale is installed so that log-diagnosing tools can function properly.
  - This was added in [PR #1031](https://github.com/Botspot/pi-apps/pull/1031)

Apt/dpkg/package functions below.

- `apt_lock_wait` - waits until apt locks are released.
- `package_info` - List everything `dpkg` knows about the specified package.
  - This retrieves a block of text from the `/var/lib/dpkg/status` file.
- `package_installed` - determine if the specified package is installed.
  - Returns an exit code of `0` if the package is installed, otherwise it returns `1`.
- `package_available` - determine if the specified package is able to be installed with `apt`.
  - This uses `grep` to search the `/var/lib/apt/lists/` folder.
  - Returns an exit code of `0` if the package was found, otherwise it returns `1`.
- `package_dependencies` - List the dependencies of a package
  - This simply isolates a line from the output of the `package_info` function.
  - This is *much* faster than doing an `apt-cache search`.
- `less_apt` - Reduce the output of an `apt` operation.
  - Example usage:
  ```bash
  sudo apt update 2>&1 | less_apt
  ```
- `apt_update` - A wrapper function to run `sudo apt update`.
  - This will wait for apt locks to be released, handle status information, and display helpful tips if packages are upgradable or autoremovable.
  - Arguments to the function will be passed on to the `apt` command.

Below are three functions that manage **the Pi-Apps local APT repository**.  
This is a special folder (`/tmp/pi-apps-local-packages`) used by the `install_packages` function to handle installing local deb files. Installing local packages from a repository improves dependency-handling, condenses the operation into one `apt` operation, and allows the packages to be specified in any order.  

- `repo_add` - Add the specified deb file(s) to the local repository.
  - This simply copies specified files to the `/tmp/pi-apps-local-packages` folder.
- `repo_referesh` - Index the local repository, create a `Packages` file, and a `source.list`.
  - At this point, you can make `apt` use the repository by passing this flag to it: `-o Dir::Etc::SourceList=/tmp/pi-apps-local-packages/source.list`
- `repo_rm` - Removes the local repository.
- `app_to_pkgname` - Convert an app-name to an `apt`-compatible package name.
  - This function generates the name to use for creating dummy apt packages. The naming scheme is: `pi-apps-XXXXXXXX` (each `X` can be any lowercase letter or number)
  - View which dummy packages are installed now by running `apt search pi-apps-` in a terminal.
- `install_packages` - Used by apps to install packages.
  - This function is replacing the `pkg-install` script.
  - Example usage: `install_packages package1 /path/to/package2.deb https://example.com/package3.deb package4-* || exit 1`
  - First, each argument is analyzed.
    - If it's a URL, the file is downloaded and added to the local repository.
    - If it's a deb-file, it's added to the local repository.
    - If it contains regex (regular expression, aka the `*` character), a list of packages is generated using the `apt-cache search` command.
  - Next, the local repository is initialized. (if necessary)
  - Now an `apt_update` takes place.
  - It's time to configure and install an empty apt-package that "depends on" the packages we want to install. We refer to it as a "dummy deb".
    - First the name of the dummy deb is determined, using the `app_to_pkgname` function.
    - If the dummy deb is *already* installed, `install_packages` will inherit its dependencies and then purge the dummy deb. This means that **the `install-packages` function can be used multiple times**in an app's script because it's accumulative.
    - The dummy deb is created, packaged, and finally installed.
- `purge-packages` - Used by apps to remove packages that they previously installed.
  - This function accepts no arguments. It reads the `$app` variable, purges its associated dummy deb, and autoremoves any packages that are no longer necessary.
- `get_icon_from_package` - Given a package (or space-separated list of packages), this function will automatically find the program icon for it.
  - This is useful for the `createapp` script to automatically find a suitable icon for package-apps you're trying to add.
  - This uses `dpkg-query` to list all files each package installed. The list is filteres to only show `.png` files in `/icons/` or `/pixmaps/` folders.
  - The list is sorted by filesize to find the picture with the most pixels.

End of apt functions. App functions below.

- `list_apps` - List all apps that match a given criteria. (In a newline-separated format)
  - `list_apps local` will list apps that exist locally.
  - `list_apps` is the same as `list_apps local`.
  - `list_apps app` will list all apps, both local and online.
  - `list_apps installed` will list apps that are currently installed.
  - `list_apps corrupted` will list apps that are currently corrupted.
  - `list_apps disabled` will list apps that are currently disabled.
  - `list_apps uninstalled` will list apps that are currently uninstalled.
  - `list_apps have_status` will list apps that currently have a known status. (A clean Pi-Apps installation won't have any status files)
  - `list_apps missing_status` will list apps that don't have status files.
  - `list_apps cpu_installable` will list apps that have an installation script compatible with your operating system's CPU architecture.
    - If "app1" only has an `install-64` script but your system is 32-bit, then "app1" will be excluded from this list.
    - Likewise, if "app1" only has an `install-32` script but your system is 64-bit, then "app1" will be excluded from this list.
  - `list_apps package` will list apps that don't have scripts but have a `packages` file.
  - `list_apps standard` will list apps that do have scripts and don't have a `packages` file.
  - `list_apps hidden` will list apps that are in the special "hidden" category.
  - `list_apps visible` will list apps that are **not** in the special "hidden" category.
  - `list_apps online` will list apps that exist in the `update/pi-apps/apps` folder.
  - `list_apps online_only` will list apps that are **only** in the `update/pi-apps/apps` folder.
  - `list_apps local_only` will list apps that are **not** in the `update/pi-apps/apps` folder.
- `list_intersect` - Takes two lists of apps and *intersects* them, meaning that only apps that are listed in **both** lists are returned.
  For example, this will show apps that are both cpu_installable and visible:
  ```
  list_apps cpu_installable | list_intersect "$(list_apps visible)"
  ```
- `list_subtract` - Takes two lists of apps and *subtracts* one from other, meaning that only apps listed in the first list and **not** in the second list, are returned.
  For example, this will show apps that are *not* compatible with your system's architecture:
  ```
  list_apps local | list_subtract "$(list_apps cpu_installable)"
  ```
- `read_category_files` - Generates a list of categories; data compiled from the `data/category-overrides` and `etc/categories` files, with added support for unlisted apps.
- `app_categories` - Format the categories file, then list all apps, as if they were inside folders, based on the categories file. Also lists all apps under special "Installed" and "All Apps" categories.

- `bitly_link` - Increase/decrease the "number of users" a certain app has.
  - Botspot creates [bitly](https://bitly.com) links for every app: one link for installing it, and one link for uninstalling it.
  - Bitly will track how many times each link has been clicked.
  - Assuming the "Enable Analytics" setting was not turned off, this function will "click" one of those links.
  - Botspot uses a script to upload bitly's statistics to the [pi-apps-analytics](https://github.com/Botspot/pi-apps-analytics) repository.
- `usercount` - returns the number of users an app has, based on the current number in the [pi-apps-analytics](https://github.com/Botspot/pi-apps-analytics) repository.
  To display the number of users for the Arduino app:
  ```
  usercount Arduino
  ```
- `script_name` - returns name of install script(s) for the specified app. Possible outputs: '', 'install', 'install-32', 'install-64', 'install-32 install-64'
  Usage:
  ```
  script_name Arduino
  ```
- `script_name_cpu` - Given an app, this returns the name of the app's installation script that would be run if you ran it.
  - For example, if your operating system is 32-bit and the app has an `install-32` script, this function would return "install-32".
  - If your operating system is 64-bit and the app has an `install-64` script, this function would return "install-64".
  - If the app has an `install` script, this function would return "install".
  - If none of the above, don't return anything.
- `app_status` - return the given app's current status.
  - If the app's status file does not exist, this function returns 'uninstalled'.
  - Otherwise, this function returns the contents of the app's status file.
- `app_type` - Determine if an app is a `standard` app or if it's a `package`.
- `will_reinstall` - Return an exit code of `0` if the specified app would be reinstalled during an update, otherwise return an exit code of `1`.
  - If the app's existing installation script is not identical to the new version of the installation script, AND the app is currently installed, exit with a code of `0`, otherwise exit `1`.
- `app_search` - Search all apps for the specified search query.
  - In each app-folder, this will search for matches in the following files:
    -  `description`
    - `credits`
    - `website`
  - It hides incompatible and invisible apps before displaying the results. (list of app names, one per line)
- `app_search_gui` - A graphical front-end for the `app_search` function.
  - This displays all results from `app_search` in a graphical list.
  - One app should be selected from the list before clicking OK.
    - If only one app is displayed in the list, no need to select it.
  - The chosen app (if any) is returned.
- `generate_app_icons` - Resize a specified image and place the icons in the specified app-folder.
  - This requires imagemagick to be installed. If it's missing, a dialog box will appear and ask permission to install it.
  - Example usage: `generate_app_icons /path/to/my-image.png my-app`
- `refresh_pkgapp_status` - For the specified package-app, if dpkg thinks it's installed, then mark it as installed.
- `refresh_all_pkgapp_status` - For every package-app, if dpkg thinks it's installed, then mark it as installed.

Logfile functions below.
- `get_logfile` - Find the most recent logfile for the specified app.
- `log_diagnose` - Search a specified logfile for phrases that indicate non-errors.
  - Many errors are not Pi-Apps's fault. Most are outside of Pi-Apps's control, but caused by user-interference, Internet problems, or apt configuration errors.
  - Errors are categorized into three types: `system`, `internet`, `package`, and unknown.
    - If a known phrase is identified, the `$error_type` variable is set to either `system` or `internet` or `package`.
    - If no phrases were identified, the `$error_type` variable will be set to `unknown`. Only when the error_type is "unknown" will Pi-Apps allow the user to send an error report.
  - Each detected error has an accompanying caption for the user to read. This caption explains what the problem is and how to fix it.
    - As multiple error messages might be identified, the error captions are stored in an **array** variable called `$error_caption`. Storing explanations in an array allows multiple explanations to be displayed to the user.
  - Before exiting, this function returns the collected information. The first line is the value of `$error_type`, while subsequent lines are the value(s) of `$error_caption`.
- `format_logfile` - Log files store the entire output of all apps being installed or uninstalled. This function formats the logfile to improve its readability.
  - Unwanted patterns are removed, like terminal color-codes, long arrays of periods, etc.
  - All instances of the `\r` character are replaced by the `\n` character.
  - A header is added to the file, containing the output from `get_device_info`.
- `send_error_report` - Sends a log file to the Pi-Apps developers.
- `send_error_report_gui` - A graphical front-end for `send_error_report` - asks the user permission before sending an error log.
  - Please note that this is currently not being used. The `manage` script has its own error-reporting gui and directly uses `send_error_report`.

Below are all functions that don't have anything to do with apps.
- `runonce` - this function runs code only once, ever. Used by other scripts to run one-time workarounds to ensure a smooth transition as Pi-Apps evolves.
  - For example, this is a real usage of `runonce` in the Pi-Apps `gui` script:
    ```bash
    #ensure curl is installed
    runonce <<"EOF"
      if ! command -v curl >/dev/null ;then
        sudo apt install -y curl
      fi
    EOF
    ```
    It installs `curl` on the system, but only tries once.
  - This works by hashing the entire command first, using `sha256sum`.
  - If the hash matches a line in the `data/runonce_hashes` file, nothing occurs. Otherwise, the command is executed.
- `text_editor` - Use a text editor to open a file.
  - This obeys your choice of "Preferred text editor".
  Usage:
  ```
  text_editor /path/to/your.file
  ```
- `view_file` - Display a maximized `yad` window to view a file. This is used to view logfiles.
- `is_supported_system` - determines if your operating system is supported. This returns an exit-code of `0` if supported, otherwise`1`.
  If any of the below criteria are true, then your system is unsupported:
  - The kernel matches "x86" or "i686 or "i386".
  - The `/proc/version` file matches "Android".
  - The operating system's `PRETTY_NAME` matches "stretch", "wheezy", "jessie", "manjaro" or "Ubuntu 16".
  - The kernel matches "armv6*".
  - The script is being run as `root`.
  - The system has less than 500MB of free space.
- `get_device_info` - summarizes the current system setup for debug use.
  - To view the output on your  system, run this command:
  ```
  ~/pi-apps/api get_device_info
  ```
  - This function is used in the `format_log_file` function.
- `functions_to_files` - Takes every function in the `api` and turns them into their own miniature bash scripts.
  - This exists purely for developer-convenience. It allows you to handle functions as if they were files.
  - It creates a folder (`~/pi-apps/function-files`) and then places files in it.
- `files_to_functions` - Takes every file in the `function-files` folder and re-combines them.
  - The resulting output is printed to the terminal.

Command interceptors below:
- `git_clone` - Wrapper function for the `git clone` command with improvements:
  - Status information is displayed. ("Downloading XXXXXX repository...")
  - `git`'s output is suppressed. But if the operation fails, its full output will be displayed in the error message.
  - Before cloning the repo, the destination folder is removed. This prevents the common error "`Fatal: destination path 'XXXXXX' already exists and is not an empty directory.`".
  - There may be times when an app-script doesn't want the output suppressed, or status information, or the folder removed first. That's why this function is an "opt-in" function; script-writers have to consciencely switch to `git_clone` if they want to.
- `wget` - This function overrides the `wget` command in all app-scripts. To speed up app-installation, it uses the `aria2c` tool when possible. Aria2c is faster and more reliable than wget, but it can't be used in all situations.
  - To determine if `aria2c` can be used, this function parses all flags given to it. It stores the url and the output filename.
    - If any flags other than `-q`, `-O`, and `-qO` are passed, `wget` is used.
    - If the `aria2c` command does not exist, `wget` is used.
    - If the output is not a file but is being sent to `stdout` (using the `-qO-` flag, for example), `wget` is used.
  - If `aria2c` is enabled, it runs with the following flags:
  `-c -x 16 -s 16 -m 10 --retry-wait 30 --console-log-level=error --show-console-readout=false --summary-interval=0 "$url" --dir '/' -o "${file:1}" --allow-overwrite`
    - If the `-q` flag was passed, the `--quiet` flag is sent to aria2c.
  - Otherwise, if aria2c was ruled out, `wget` is run as it normally would.
  - This function is designed to operate seamlessly in 100% of cases. App-developers *should not have to even know* that this function is really translating `wget` commands to `aria2c` - it should operate exactly the same as `wget`, but faster.
- `chmod` - Wrapper function for the `chmod` command with status information.
  - This displays "`Making executable: /path/to/file`".
- `unzip` - Wrapper function for the `unzip` command with status information.
  - This displays "`Extracting: /path/to/file`".

And with that, we come to the end of functions in the `api` script. If you see a function that's not listed here, please let us know.