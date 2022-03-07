---
template: overrides/main.html
---

#### Location:
On a default pi-apps installation, you will find this script at `/home/pi/pi-apps/etc/categoryedit`.  
This file is located in the `etc` folder because it's one of the smaller scripts.
#### Purpose:
Manage Pi-Apps categories.  
As of commit https://github.com/Botspot/pi-apps/commit/ab1fcb5a114ab720df1712aaf151a9733e18a94c, there are *two* categories files: a local one and a global one. The global file is kept updated like any other file, while the local file is empty by default but can contain **overrides**.  
The global categories file: `/home/pi/pi-apps/etc/categories`  
The local categories file: `/home/pi/pi-apps/data/category-overrides`  
#### Usage:
Run the category editor:
```bash
~/pi-apps/etc/categoryedit
```
Move the Arduino app to the Internet category:
```bash
~/pi-apps/etc/categoryedit Arduino Internet
```
Move the Arduino app to the top level (no category):
```bash
~/pi-apps/etc/categoryedit Arduino
```