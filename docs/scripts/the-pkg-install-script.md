## The `pkg-install` script
#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/pkg-install`. 
#### Purpose:
Some background information first:  
- Goal: Pi-Apps is designed for people who install an app, try it out, then later uninstall it. You should not have to think twice before installing an app. Users should have confidence that **uninstalling the app will undo all changes and restore all disk-space.**
- Problem: Many apps need to install apt **packages** in order to work. On the surface, this does not seem like a big problem at all: if "app1" installs "package1", "package2", and "package3", then those packages should be purged while uninstalling "app1". What's the problem with that? **Dependencies.**  
What if some other utility requires "package1" to function? Now that you uninstalled "app1", "package1" just got uninstalled.
  - Best-case scenario: that utility will not work anymore.
  - Worst-case scenario: you just broke an essential part of your system and it will fail to boot.
- Solution: When uninstalling an app, **only remove packages that are not required by anything else**. To accomplish this, we can't just install packages the normal way with `sudo apt install`. Instead, we need to generate a **dummy deb** - a custom apt package that lists "package1", "package2", and "package3" as **dependencies**. Later, when the app is being uninstalled, the dummy deb is removed and a simple `apt autoremove` is enough to safely remove the packages.
#### Usage:
In an app's `install` script, there may be a line like this to install "package1", "package2", and "package3":
```bash
DIRECTORY=$HOME/pi-apps
"${DIRECTORY}/pkg-install" 'package1 package2 package3' "$(dirname "$0")" || exit 1
```
This is equivalent to:
```bash
sudo apt update --allow-releaseinfo-change
sudo apt install -yf --no-install-recommends --allow-downgrades package1 package2 package3
```
#### How it works:
1. First, `pkg-install` sets the language variables to `C`. This ensures that apt's output is parsed correctly, even if the system is using a different language,
2. `pkg-install` runs `sudo apt update`.
3. The output of `sudo apt update` is parsed for various messages from `apt`.
    - If the output contains "autoremove to remove them", you will receive a message that some packages can be removed with `apt autoremove`.
    - If the output contains "packages can be upgraded", you will receive a message that some packages can be upgraded.
    - If the output contains "W:" or "E:" or "Err:", `pkg-install` will **exit with an error** saying that your apt system is messed up.
    The exact error message depends on apt's exact output - it is designed to help users navigate through apt errors and provides instructions for how to sign a repository, remove a broken repository, or check for an Internet connection.
4. If any filenames (paths to a local deb package) are specified, `pkg-install` will install each one, then mark it as autoremovable. (Using this command: `sudo apt-mark auto "$packagename"`
5. If any package names include **regular expression**, `pkg-install` expands the names with `apt-cache search`.
6. Finally, the dummy deb is created. As mentioned earlier, this package will list the desired packages as dependencies.
    - The dummy deb for each app has to be named something unique. But this poses a problem because apps can have space characters while apt does not support space characters.
    - This problem is resolved by naming each dummy-deb based on a **hash of its name**.
    - The code used to do this is:
   ```bash
   echo -n 'pi-apps-' ; echo "$app" | md5sum | cut -c1-8 | awk '{print $1}'
   ```
    - Feel free to replace `"$app"` with an app-name of your choice to see what its package name would be.
7. If the dummy deb's name is already installed, purge it and then continue.
8. Finally, the dummy deb is installed with `apt`. All packages mentioned in the "`Depends:`" field of the dummy deb are installed as a dependency of the dummy deb.
9. If apt fails, its errors are diagnosed in the same way errors were diagnosed earlier when `sudo apt update` was run.
10. `pkg-install` exits with a code of `0` if everything was successful, otherwise it exits with a code of `1`.