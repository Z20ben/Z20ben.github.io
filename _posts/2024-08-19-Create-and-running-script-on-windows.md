---
title: "Create and Running Script on Windows with Task Scheduler and Regedit"
description: Example how to create and running script on windows platform
date: 2024-08-19 00:00:00 +0700
comments: true
categories: [Script]
tags: [script, bash, regedit, task scheduler]
pin: true
mermaid: true
image:
  path: /assets/img/project-build-a-task-scheduler-using-bash.png
  lqip: data:image/webp;base64,UklGRmQAAABXRUJQVlA4IFgAAACwAwCdASoUAAoAPzmEuVOvKKWisAgB4CcJagCdACPtc2cdt/rdwAD8sXh+QJz8TmYpoFIBkx23Edrh4ORyG3MHiKF154KIg3r3/2kkDMimgWpzj3VnonAA
  alt: Build a task scheduler using bash.
---

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

Welcome to this tutorial where we will explore the powerful features of Windows in automating tasks and managing system settings through scripts. In this guide, we will walk you through the process of creating a script, setting it up to run automatically using Windows Task Scheduler, and making necessary adjustments through the Windows Registry Editor (RegEdit).

## Prerequisites
To follow this tutorial, you will need :
1. Basic understanding of Bash scripting.
2. Access to a terminal.

## Objectives
1. Automation of Task Execution
2. Time-Based Scheduling
3. Continuous Monitoring
4. Feedback and Status Reporting
5. Concurrency and Overlapping Tasks


## Task 1 : Set Task Scheduler
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


## Task 2 : Set Regedit
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
{: file='Notes'}

Example Command:

Here is the exact command you should enter in the registry entry:
```bash
schtasks /run /tn "HjScript\HjScript"
```
{: file='C:\Path\To\Your\Script.bat'}

## Conclusion

This basic task scheduler demonstrates how to use Bash scripting to automate tasks by reading from a configuration file. With further development, it can be extended to handle complex scheduling and task management features. Happy scripting!
