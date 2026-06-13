# Linux Task 03: Process Management, System Monitoring & Basic Shell Scripting

## Objective
The purpose of this task is to understand how Linux manages processes, monitors system resources, and automates tasks using shell scripts.

These are essential skills for Linux Administrators, SOC Analysts, and Cyber Security Professionals.

---

## Table of Contents
- [Part A: Process Monitoring](#part-a-process-monitoring)
- [Part B: Process Management](#part-b-process-management)
- [Part C: System Monitoring](#part-c-system-monitoring)
- [Part D: Service Monitoring](#part-d-service-monitoring)
- [Part E: Shell Scripting](#part-e-shell-scripting)
- [Part F: Security Monitoring Challenge](#part-f-security-monitoring-challenge)
- [Part G: Mini SOC Activity](#part-g-mini-soc-activity)
- [Command Outputs](#command-outputs)

---

## Part A: Process Monitoring

### Commands Run
- `ps`
- `ps aux`
- `top`
- `htop`

> If htop is not installed: `sudo apt install htop`

### Screenshots
- `Screenshots/Part-A_process_monitoring_1.png` (ps, ps aux)
- `Screenshots/Part-A_process_monitoring_2.png` (top)
- `Screenshots/Part-A_process_monitoring_3.png` (htop)

### Questions & Answers

**1. What is a Process?**

> A process is a running instance of a program. When a program is executed, the operating system loads it into memory and allocates it resources (CPU time, memory, files) so it can run. Each process operates independently and has its own memory space.

**2. What is a PID?**

> A PID (Process ID) is a unique number assigned by the Linux kernel to every running process. It is used to identify, monitor, and manage processes (for example, when sending signals like `kill`).

**3. Which process is consuming the most CPU?**

> Based on the `top`/`htop` output, **xfwm4** (PID 1244) was using the highest CPU at around 1.4%, followed by **Xorg** (PID 786) at around 0.7%. Overall CPU usage on the system was very low (mostly idle).

**4. Which process is consuming the most Memory?**

> **Xorg** (PID 786) was consuming the most memory at around 3.6% (RES ~142 MB), followed by **xfwm4** (PID 1244) at around 3.5%.

---

## Part B: Process Management

### Steps
1. Opened a new terminal and ran: `sleep 300`
2. Found the process using: `ps aux | grep sleep`
3. Terminated the process using: `kill 306812`
4. Ran `sleep 300` again, then terminated it forcefully using: `kill -9 307524`

### Documentation

| Detail        | Value |
|----------------|-------|
| PID found      | 306812 (first attempt), 307524 (second attempt) |
| Command used   | `kill 306812` then `kill -9 307524` |
| Result         | The first `sleep 300` was stopped with a normal `kill`, and the terminal reported `zsh: terminated  sleep 300`. After running `sleep 300` again and using `kill -9` on its PID, the terminal reported `zsh: killed  sleep 300`. The `kill -9` (SIGKILL) immediately and forcefully ended the process without allowing it any cleanup, whereas the plain `kill` (SIGTERM) allowed a normal termination. |

### Screenshots
- `Screenshots/Part-B_Process_management.png`

---

## Part C: System Monitoring

### Commands Run
- `free -h`
- `df -h`
- `uptime`
- `uname -a`

### Recorded System Information

| Metric           | Value |
|-------------------|-------|
| Total RAM         | 3.8 GiB |
| Available RAM     | 2.6 GiB |
| Disk Usage        | 19G used / 38G total on `/dev/sda1` (52% used, 17G available) |
| System Uptime     | 8 hours 43 minutes (1 user logged in, load average: 0.11, 0.05, 0.01) |
| Kernel Version    | Linux kali 6.18.12+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.18.12-1kali1 (2026-02-25) x86_64 GNU/Linux |

### Screenshots
- `Screenshots/Part-C_System_monitoring.png`

### System Summary Report

> The system (hostname: kali) has a total of 3.8 GiB of RAM, with about 2.6 GiB available and only 1.2 GiB in use, indicating low memory pressure. The root filesystem (`/dev/sda1`, 38G) is 52% used, with 17G free. The system has been up for 8 hours and 43 minutes with a single user logged in and a very low load average (0.11, 0.05, 0.01), showing the system is lightly loaded. It is running kernel version 6.18.12+kali-amd64 on a 64-bit (x86_64) architecture.

---

## Part D: Service Monitoring

### Commands Run
- `systemctl status ssh`
- `systemctl status NetworkManager`

> If a service is unavailable, an alternative available service was used.

> Note: `ssh.service` was found to be **inactive (dead)** and disabled on this system, while `NetworkManager.service` was **active (running)**, so both outputs were documented to show the difference between a stopped and a running service.

### Questions & Answers

**1. What is a Service?**

> A service (also called a daemon) is a background process managed by the system's init/service manager (systemd) that typically starts automatically at boot or on demand, and runs without direct user interaction — for example, SSH, NetworkManager, or a web server.

**2. Why are services important?**

> Services provide essential ongoing functionality such as networking, remote access, logging, and security monitoring. They allow the system to perform tasks automatically and continuously without requiring a user to manually start programs each time.

**3. How can a stopped service affect a system?**

> If a critical service is stopped, the functionality it provides becomes unavailable. For example, the `ssh.service` being inactive means no one can connect to this machine remotely via SSH. If `NetworkManager` were stopped, the system could lose network connectivity entirely, affecting internet access, DNS resolution, and other dependent services.

### Screenshots
- `Screenshots/Part-D_Service_monitoring.png`

---

## Part E: Shell Scripting

### Script: `system_report.sh`

The script displays:
- Current User
- Hostname
- Date & Time
- Current Directory
- Available Memory
- Disk Usage

See [`system_report.sh`](./system_report.sh) for the full script.

### Execution Steps
```bash
chmod +x system_report.sh
./system_report.sh
```

### Screenshots
- `Screenshots/Part-E_Shell_scripting_1.png` (script execution & output)
- `Screenshots/Part-E_Shell_scripting_2.png` (script source code in nano)

---

## Part F: Security Monitoring Challenge

### Command Research

#### 1. `netstat`

| | |
|---|---|
| Purpose | Displays network connections, listening ports, routing tables, and interface statistics. The `-tuln` flags show TCP/UDP listening sockets in numeric form without resolving hostnames. |
| Example Output | `udp6   0   0 fe80::aa00:27ff:fe6d:546   :::*` — shows an open UDP6 socket bound to a link-local address with no foreign address connected (a listening socket). |
| Security Use Case | Helps identify unexpected open ports or services listening on the network, which could indicate misconfigurations, unauthorized services, or malware establishing a backdoor. |

#### 2. `ss`

| | |
|---|---|
| Purpose | A modern, faster replacement for `netstat` used to display socket statistics — active connections, listening ports, and their states. |
| Example Output | `udp   UNCONN  0  0  fe80::a00:27ff:fe6d:546%eth0:546   0.0.0.0:*` — shows an active UDP socket in an unconnected (listening) state, similar to the netstat output above but in `ss`'s more compact format. |
| Security Use Case | Quickly auditing which ports/services are exposed on the host, useful for spotting unauthorized listeners or confirming that only expected services are accessible. |

#### 3. `who`

| | |
|---|---|
| Purpose | Shows who is currently logged into the system, along with their terminal, login time, and session. |
| Example Output | `mark   seat0   2026-06-11 20:18 (:0)` — shows user "mark" logged in on seat0 (local session) since 11 June at 20:18. |
| Security Use Case | Helps detect unexpected or unauthorized active sessions on a system, which could indicate someone else is logged in. |

#### 4. `w`

| | |
|---|---|
| Purpose | Shows currently logged-in users along with what they are doing, including login time, idle time, and CPU usage of their processes (more detailed than `who`). |
| Example Output | `mark   -   Thu20   0.00s  0.06s  lightdm --session-child 13` — shows user "mark" logged in since Thursday at 20:00, with minimal CPU usage, running a session under lightdm. |
| Security Use Case | Useful for spotting suspicious user activity in real time — e.g., a user logged in from an unexpected terminal or running unusual commands, which could indicate compromise. |

#### 5. `last`

| | |
|---|---|
| Purpose | Displays a history of user logins and logouts, reboots, and system run-level changes, sourced from the `wtmp` log file. |
| Example Output | `mark   tty7   :0   Thu Jun 11 20:18 - still logged in` and `mark   tty7   :0   Wed Jun 10 18:21 - still logged in` — shows login history for user "mark" across two sessions, both still active. |
| Security Use Case | Helps an analyst review login history to identify unusual login times, repeated failed logins, or logins from unexpected sources — useful for detecting unauthorized access attempts. |

### Screenshots
- `Screenshots/Part-_F_Security_monitoring_challenge.png` (covers `netstat -tuln`, `who`, `w`, and `last` output)

---

## Part G: Mini SOC Activity

**Scenario:** A system is running slowly. As a Security Analyst, using the commands learned today:

**1. How would you identify resource-heavy processes?**

> I would use `top` or `htop` to get a live view of CPU and memory usage sorted by resource consumption, and `ps aux --sort=-%cpu` or `ps aux --sort=-%mem` to list processes consuming the most CPU or memory. This quickly highlights which process(es) are responsible for the slowdown.

**2. How would you determine whether a process is suspicious?**

> I would check the process name and command path (is it running from an unusual location like `/tmp`?), the user that owns it (is a normal user running something with root privileges unexpectedly?), its PID/PPID relationship (does it have a strange parent process?), and whether it is consuming abnormal amounts of CPU, memory, or network resources. I would also check `netstat`/`ss` to see if the process has unexpected open network connections, and search the process name online if it's unfamiliar.

**3. What information would you collect before terminating a process?**

> Before killing a process, I would record its PID, PPID, the full command/path it was launched from, the user running it, its CPU/memory usage, any open files or network connections (via `lsof` or `ss`), and how long it has been running. This information helps with incident documentation, root-cause analysis, and determining whether the issue (or threat) might recur after the process is terminated.

---

## Command Outputs

### Part A: Process Monitoring

**`ps`**
```
  PID TTY          TIME CMD
251562 pts/0    00:00:00 zsh
252112 pts/0    00:00:00 ps
```

**`ps aux` (sample)**
```
USER   PID %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
root     1  0.1  0.4  26360 16232 ?  Ss   12:58   0:33 /usr/lib/systemd/systemd
root     2  0.0  0.0      0     0 ?  S    12:58   0:00 [kthreadd]
root   786  0.7  3.6 441804 142692 ? S    19:16   5:52 Xorg
mark  1244  0.7  3.5 1354640 139048 ? S  19:16   2:19 xfwm4
```

**`top` (header)**
```
top - 21:20:01 up  8:21,  1 user, load average: 0.04, 0.04, 0.01
Tasks: 208 total, 1 running, 207 sleeping, 0 stopped, 0 zombie
%Cpu(s):  0.1 us,  0.4 sy,  0.0 ni, 99.5 id
MiB Mem :  3919.4 total,   227.2 free,  1191.7 used,  2794.1 buff/cache
MiB Swap: 2135.0 total,  2134.0 free,     1.0 used
```

**`htop` (top entries)**
```
PID  USER  PRI  NI  VIRT  RES  CPU%  MEM%  TIME+    Command
1244 mark   20   0 1322M 135M   1.4   3.5  2:19.59  xfwm4
786  root   20   0 431M  139M   0.7   3.6  5:42.16  /usr/lib/xorg/Xorg
1366 root   20   0 315M  65M    0.7   1.6  1:10.45  xfce4/panel
```

---

### Part B: Process Management

```
$ sleep 300
$ sleep 300
zsh: terminated  sleep 300

$ sleep 300
zsh: killed     sleep 300

$ ps aux | grep sleep
mark  306812  0.0  0.0  5628 2560 pts/0  S+  21:37  0:00 sleep 300
mark  306974  0.0  0.0  6556 2876 pts/1  S+  21:37  0:00 grep --color=auto sleep

$ kill 306812

$ ps aux | grep sleep
mark  307524  0.0  0.0  5628 2560 pts/0  S+  21:38  0:00 sleep 300
mark  307977  0.0  0.0  6556 2904 pts/1  S+  21:39  0:00 grep --color=auto sleep

$ kill -9 307524
```

---

### Part C: System Monitoring

**`free -h`**
```
              total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.2Gi       365Mi        12Mi       2.5Gi       2.6Gi
Swap:          2.1Gi       1.0Mi       2.1Gi
```

**`df -h`**
```
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           392M 1008K  391M   1% /run
/dev/sda1        38G   19G   17G  52% /
tmpfs           2.0G  4.0K  2.0G   1% /dev/shm
tmpfs           2.0G   16K  2.0G   1% /tmp
none            1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs           392M  112K  392M   1% /run/user/1000
none            1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
```

**`uptime`**
```
21:42:24 up  8:43,  1 user,  load average: 0.11, 0.05, 0.01
```

**`uname -a`**
```
Linux kali 6.18.12+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.18.12-1kali1 (2026-02-25) x86_64 GNU/Linux
```

---

### Part D: Service Monitoring

**`systemctl status ssh`**
```
○ ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:sshd(8)
             man:sshd_config(5)
```

**`systemctl status NetworkManager`**
```
● NetworkManager.service - Network Manager
     Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-06-12 16:40:26 IST; 1 day 5h ago
   Main PID: 160105
      Tasks: 4 (limit: 4456)
     Memory: 9.3M (peak: 10.1M)
        CPU: 673ms
     CGroup: /system.slice/NetworkManager.service
             └─160105 /usr/sbin/NetworkManager --no-daemon
```

---

### Part E: Shell Scripting

**`./system_report.sh` output**
```
System Information Report
--------------------------
User: mark
Hostname: kali
Date: Saturday 13 June 2026 09:53:19 PM IST
Current Directory: /home/mark

Memory Usage:
              total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.3Gi       312Mi        11Mi       2.6Gi       2.6Gi
Swap:          2.1Gi       1.0Mi       2.1Gi

Disk Usage:
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           392M 1008K  391M   1% /run
/dev/sda1        38G   19G   17G  52% /
tmpfs           2.0G  4.0K  2.0G   1% /dev/shm
tmpfs           2.0G   16K  2.0G   1% /tmp
none            1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs           392M  112K  392M   1% /run/user/1000
none            1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
```

---

### Part F: Security Monitoring Challenge

**`netstat -tuln`**
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp6       0      0 fe80::a00:27ff:fe6d:546 :::*
```

**`ss -tuln`**
```
Netid State    Recv-Q Send-Q  Local Address:Port   Peer Address:Port
udp   UNCONN   0      0       fe80::a00:27ff:fe6d:546%eth0:546   0.0.0.0:*
```

**`who`**
```
mark     seat0        2026-06-11 20:18 (:0)
```

**`w`**
```
 21:56:04 up  8:57,  1 user,  load average: 0.10, 0.08, 0.01
USER  TTY  FROM    LOGIN@  IDLE   JCPU   PCPU  WHAT
mark  -            Thu20   0.00s  0.06s  lightdm --session-child 13
```

**`last`**
```
mark     tty7         :0               Thu Jun 11 20:18   still logged in
mark     tty7         :0               Wed Jun 10 18:21   still logged in

wtmpdb begins Wed Jun 10 18:21:47 2026
```
