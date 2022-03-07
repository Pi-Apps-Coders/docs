---
template: overrides/main.html
---

The Pi-Apps folder contains several main subfolders. These folders have special characteristics and are treated differently during updates.
- The `data` folder and `logs` folder contain files that should ***never be updated***. They contains settings, cached files, app-status files, log files, etc.
- The `apps` folder contains apps. During an update it this folder is handled differently too.
- Other folders like `icons` and `etc` are handled the same as individual files in the main pi-apps folder. 