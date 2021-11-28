#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/createapp`. 
#### Purpose:
A GUI to help you create/edit an app. 
#### Usage:
To create a new app:
```bash
~/pi-apps/createapp
```
To edit an existing app:
```bash
~/pi-apps/createapp Arduino
```
#### How it works:
This script is one long `while` loop with a `case` statement for each step. (similar to the `gui` script)  
Each dialog allows you to go forward or backward and the current step is stored in the `$step` variable.  
That's about it. View the script to see how each step works.