# Linux-Persistence-Mechanisms-Cron-XDG-Autostart# Linux Persistence Mechanisms: Cron & XDG Autostart

**A hands-on Linux security assessment exploring persistence mechanisms, MITRE ATT&CK technique emulation, and defensive detection opportunities.**

![Linux](https://img.shields.io/badge/Platform-Linux-black?logo=linux)
![Python](https://img.shields.io/badge/Language-Python-blue?logo=python)
![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-red)
![Security](https://img.shields.io/badge/Focus-Persistence%20%26%20Detection-orange)

---

## Project Overview

Persistence is an important aspect of cybersecurity because attackers may attempt to maintain access to a compromised system even after their initial interaction with it.

Linux provides legitimate mechanisms for automating tasks and launching applications. However, these features can also be abused to establish persistence.

In this project, I conducted a controlled practical assessment to demonstrate two Linux persistence mechanisms:

- **Cron-based persistence (MITRE ATT&CK T1053.003)**
- **XDG Autostart persistence (MITRE ATT&CK T1547.013)**

The practical was conducted in a disposable Linux virtual machine using a harmless Python timestamp marker.

The objective was to observe automatic execution, examine available detection opportunities, and verify successful removal of both persistence configurations.

## 1. The Scenario

### Client: NexaCore Technologies Ltd. (Fictional)

Imagine managing the security of a technology company where employees use Linux workstations every day.

Everything appears normal. Employees log in, launch their applications, perform their daily activities, and shut down their systems.

But what if a program could continue executing without an employee manually opening it?

What if a script could be configured to run every minute, every hour, or at a specific time without attracting the user's attention?

And what if another program could launch automatically every time an employee logs into their Linux desktop?

These were the questions raised by NexaCore Technologies' IT department after observing unusual application execution behaviour on some of its Linux workstations.

The IT team wanted to understand whether legitimate Linux scheduling and desktop startup mechanisms could be abused to maintain persistence.

As part of a controlled security assessment, I was tasked with demonstrating two Linux persistence mechanisms, understanding their execution behaviour, and identifying opportunities for detection and response.

The central security question was:

**If an attacker used legitimate Linux startup and scheduling mechanisms to maintain persistence, would a defender know where to look?**

## 2. Assessment Objectives

The assessment focused on:

1. Demonstrating scheduled execution using Cron.
2. Demonstrating login-triggered execution using XDG Autostart.
3. Mapping both techniques to MITRE ATT&CK.
4. Identifying useful detection opportunities and telemetry gaps.
5. Removing the persistence configurations and verifying cleanup.

## 3. Lab Environment

| Component | Description |
|---|---|
| Operating System | Linux kali 6.8.11-amd64 |
| Virtualization | Disposable Linux VM |
| Programming Language | Python 3 |
| Scheduling Mechanism | Cron |
| Desktop Startup Mechanism | XDG Autostart |
| Evidence Collection | Python timestamp marker and system logs |

All activities were performed within an authorized, disposable virtual machine using a normal user account.

A VM snapshot was taken before the practical to support recovery.

## 4. Technique One: Cron-Based Persistence

**MITRE ATT&CK: T1053.003 – Scheduled Task/Job: Cron**

Cron is a time-based job scheduler available on Unix-like operating systems. It allows commands and scripts to execute automatically according to a defined schedule.

### Practical Demonstration

I configured a labelled cron entry to execute a harmless Python marker every minute.

The script recorded a UTC timestamp and the execution label in a local JSONL file.

### Execution Evidence

The following screenshot shows the configured cron entry.

![Cron Configuration](evidence/cron/crontab-entry.png)

The following evidence captures recurring executions recorded by the Python marker.

![Cron Execution](evidence/cron/cron-execution.png)

### Observation

The marker recorded recurring executions without requiring manual interaction.

This demonstrated how a scheduled task could continue executing in the background under the configured user's account.

### Detection Opportunities

Defenders can investigate:

- Changes to user crontabs.
- Cron-related system logs.
- Scheduled command execution.
- Process creation and parent-child relationships.
- Unexpected or unauthorized scheduled entries.

## 5. Technique Two: XDG Autostart Persistence

**MITRE ATT&CK: T1547.013 – Boot or Logon Autostart Execution: XDG Autostart Entries**

XDG Autostart allows applications to launch automatically when a user logs into a compatible graphical desktop environment.

It uses `.desktop` files to define applications and commands that should execute during desktop startup.

### Practical Demonstration

I created a desktop entry under:

`~/.config/autostart/`

The entry configured the Python marker to execute when the user logged into the graphical desktop session.

### Configuration Evidence

The following screenshot shows the XDG Autostart desktop entry.

![XDG Autostart Configuration](evidence/xdg-autostart/desktop-entry.png)

### Execution Evidence

After the desktop login test, the Python marker recorded an execution labelled `autostart`.

![XDG Autostart Execution](evidence/xdg-autostart/autostart-execution.png)

### Detection Opportunities

Defenders should investigate:

- Newly created `.desktop` files.
- Unexpected modifications in the autostart directory.
- Unusual commands specified in the `Exec` directive.
- Processes launched during desktop login.
- The relationship between login events and application execution.

## 6. Emulation and Detection Analysis

### Hypothesis

Before installing the cron persistence mechanism, I predicted that the marker would append a new record containing the `cron` execution label every minute.

I also expected relevant process execution telemetry to show the scheduled execution chain.

### Observed Results

The marker file recorded recurring minute-by-minute execution without manual interaction.

Cron-related PAM session-opening records were also observed, with timestamps corresponding to the marker activity.

The XDG Autostart test also produced an `autostart` execution record in the marker file.

These observations supported the execution hypothesis. However, direct process execution telemetry would provide stronger confirmation of the complete process chain.

### False Positive Example

Legitimate desktop applications such as Dropbox, Discord, or Redshift may create autostart entries during installation.

Consequently, not every new `.desktop` file represents malicious persistence.

### Collection Gap

Without process auditing or endpoint monitoring, defenders may lack detailed visibility into executed commands and their parent processes.

Standard cron logs may show that a job ran without providing complete process execution details.

### Recommended Improvement

Deploy auditd or an appropriate Endpoint Detection and Response (EDR) solution to monitor:

- Cron configuration changes.
- File creation and modification within XDG Autostart directories.
- Process execution and command-line arguments.
- Relevant parent-child process relationships.

### Supporting Evidence

![Cron Journal Evidence](evidence/detection/cron-journal.png)

## 7. Objective-Led Red Teaming vs Vulnerability Listing

Vulnerability listing focuses on identifying weaknesses such as outdated software and system misconfigurations.

Objective-led red teaming goes further by emulating realistic adversary behaviour to assess whether defensive controls can identify and respond to it.

This practical focused on a specific objective: demonstrating persistence through legitimate Linux mechanisms and examining the evidence available to defenders.

## 8. Command and Control Context

In a real-world attack, a persistence mechanism could be configured to launch a malicious payload or command-and-control agent.

Such an agent might communicate with an external command-and-control server over HTTP, HTTPS, or DNS.

This practical did not deploy a C2 implant or establish external communication.

The supplied proxy evidence was used only to discuss the concept.

## 9. Cleanup and Verification

After completing the demonstrations, I removed the labelled cron entry and the XDG Autostart desktop file.

I subsequently monitored the system, waited for scheduled execution intervals, and performed another desktop login.

The marker file showed no new execution records after the final recorded pre-removal marker at 10:59:01 UTC.

Cleanup was recorded at 14:10:47 UTC.

Historical evidence was preserved for assessment.

### Cleanup Evidence

![Cleanup Verification](evidence/cleanup/cleanup-verification.png)

## 10. Key Takeaways

This exercise improved my understanding of:

- Linux persistence mechanisms.
- Scheduled and login-triggered execution.
- MITRE ATT&CK technique mapping.
- Security telemetry and detection opportunities.
- False positives and collection gaps.
- Safe technique emulation and cleanup verification.

**The key lesson:**

Understanding how persistence works is only one part of the job. Knowing where to look for evidence of it is equally important.

## 11. Project Structure

```text
linux-persistence-detection-lab/
│
├── README.md
│
├── docs/
│   └── assessment-report.md
│
├── scripts/
│   └── marker.py
│
└── evidence/
    ├── baseline/
    │   └── vm-snapshot.png
    │
    ├── cron/
    │   ├── crontab-entry.png
    │   └── cron-execution.png
    │
    ├── xdg-autostart/
    │   ├── desktop-entry.png
    │   └── autostart-execution.png
    │
    ├── detection/
    │   └── cron-journal.png
    │
    └── cleanup/
        └── cleanup-verification.png
