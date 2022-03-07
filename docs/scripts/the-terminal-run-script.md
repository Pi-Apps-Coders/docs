---
template: overrides/main.html
---

#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/etc/terminal-run`.  
This file is located in the `etc` folder because it's one of the smaller scripts.
#### Purpose:
This script may be Botspot's *greatest contribution to the open-source world*. Apart from this script, there is no reliable way to:
- Run a *newline-separated* list of commands in a new terminal window.
- Set the terminal's title, as desired.
- Keep the script running until the terminal has been closed.
- Support a variety of terminals.
#### Usage:
This command will run two commands in a terminal and set the terminal's title to "Upgrading your packages":
```bash
~/pi-apps/etc/terminal-run 'sudo apt update
sudo apt upgrade' "Upgrading your packages"
```
The `terminal-run` script will not exit until the terminal closes.
#### How it works:
As mentioned earlier, this script supports running a ***list*** of commands in a terminal. That may sound easy enough, but it's not.
- Terminals are not designed to support this. Most terminals can, with some clever tricks, but it varies on a case-by-case basis.

As mentioned earlier, this script will keep running until the terminal has been closed. How is that possible?
- Before running any commands in a terminal, a special file is created in the `/tmp` folder. This file is empty for now.
- Then, the terminal is instructed to write its PID (process ID) *to the file*.
- Now, while the terminal is running, the `terminal-run` script can regularly check if that PID process is still running.
  - It does this by periodically checking if the `/proc/${PID}` folder exists.
- When the PID stops, it indicates that the terminal has been closed. Now the `terminal-run` script exits.

Supported terminals:
- lxterminal
- xfce4-terminal
- mate-terminal
- xterm
- konsole
- terminator
- gnome-terminal
- x-terminal-emulator

Debug mode: if you set the `DEBUG` variable to `1`, `terminal-run` will output the name of the currently-used terminal.
- This is useful for debugging, when no terminal appears and we want to know which terminal was being used.