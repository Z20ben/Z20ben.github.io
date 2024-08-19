---
title: "Create and running script on windows with task scheduler and regedit"
description: Example how to create and running script on windows platform
date: 2024-08-19 00:00:00 +0700
categories: [Script]
tags: [script, bash, regedit, task scheduler]
pin: true
mermaid: true
image:
  path: /assets/img/devices-mockup.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
## Set Task Scheduler
{: .mt-4 .mb-4 }

1. Set Task Scheduler
2. Open Task Scheduler Library
3. Create New Folder
4. Make name of the folder, ex : "MyScript"
5. Create Basic Task, as always
6. Make the enable history for this task [in right pane of task] to monitor task
7. Properties the Task was created ... MyScript

- General Tab
  - [x] Run only when user is logged on
  - [x] Run with highest priveleges

### Configure for Windows 10
- Triggers Tab
  - [x] [hapus saja] ganti dengan regedit diakhir
  - [x] At Startup -> [v] Add delay for 1 minutes -> [v] Enabled

- Action Tab
  - [x] Start a program => full dir of batch file => start in full path (w/o file name and w/o "")

- Condition Tab
  - [x] Power [unthick]
  - [x] Network => [v] Start only if the following network connection -> any connection

- Settings Tab
  - [x] Allow task to be run on demand
  - [x] if the tas fails, restart every -> 1 minute
    attempt to restart up to -> 5 times
  - [x] if the running task does not end when requested, force it to stop


##  Set Regedit
{: .mt-4 .mb-4 }

- Open regedit.exe.
  - [x] Navigate to HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run.
  - [x] Right-click on the right pane, select New -> String Value.
  - [x] Name the new string value as you prefer, for example, RunHjScript.
  - [x] Double-click the new string value and set its value data to schtasks /run /tn "HjScript\HjScript".

Example:

Assuming your batch file is C:\Path\To\Your\Script.bat and the task is correctly set up in Task Scheduler.
Verifying the Setup:

    Check the Task Scheduler:
        Ensure that the task is enabled and can run manually without issues.
        Right-click on the task "HjScript" in the folder "HjScript" and select "Run" to verify it runs correctly.

    Add the Registry Entry:
        Open regedit.exe.
        Navigate to HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run.
        Add a new string value with the following details:
            Name: RunHjScript
            Value: schtasks /run /tn "HjScript\HjScript"

Example Command:

Here's the exact command you should enter in the registry entry:
```bash
schtasks /run /tn "HjScript\HjScript"
```

