It makes sense to start here. After all, what is an app store without apps?  
Basically, for every app, there is a **folder**. This folder is called an "**app-folder**". On a default pi-apps installation, all app-folders are located in `/home/pi/pi-apps/apps`.  
The folder's name is the app's name. The Arduino app is a folder located at `apps/Arduino`.  
There are a few files in each app-folder:
- `install` - This is a bash script to install the app.
  - Naming the script "install" indicates that it is compatible with 32-bit and 64-bit CPU architectures.
- `install-32` - This is a bash script to install the app on 32-bit operating systems.
  - Naming the script "install-32" indicates that it is designed for the 32-bit CPU architecture only.
  - If no "install-64" script exists, **then this app will only be displayed on 32-bit systems.**
- `install-64` - This is a bash script to install the app on 64-bit operating systems.
  - Naming the script "install-64" indicates that it is designed for the 64-bit CPU architecture only.
  - If no "install-32" script exists, then **this app will only be displayed on 64-bit systems.**
- `uninstall` - This is a bash script to uninstall the app.
- `icon-24.png` and `icon-64.png` - These are the app's icons.
- `description` - This is a text file to explain what the app is, how it works, and why anybody would want it.
  - The first line of this file is used as the **tooltip** (mouse hover-text) in the list of apps.
- `credits` - This is a text file to give credit to the person who made the app.
- `website` - This is a text file containing the website URL of a given project.
  - Usually it points to a website where users can learn more about the app.