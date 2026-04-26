# 1. Window Logging for SOC
## a.Introduction
  - SOC analysts spend most of their time triaging alerts and hunting threats using logs from a SIEM system. To distinguish between normal and malicious activity, analysts must deeply understand logs—how they appear, how to interpret them, and what suspicious behavior they may reveal.
  - This module introduces Windows logging, which is a critical skill for SOC analysts and DFIR professionals.

  - Learning Objectives
    - Understand how to locate and interpret key Windows Event Logs
    - Learn to monitor important log sources such as:
      - Sysmon
      - PowerShell logs
    - Prepare for hands-on practice in SOC-SIM environments
    - Practice analyzing logs across multiple datasets

  - Recommended Preparation
    - Review Log Fundamentals
    - Learn and practice Sysmon
    - Understand how to query Windows Event Logs
    - Study core Windows processes (to recognize abnormal behavior)
## b. Logging
- Every action on your system (login, file creation, running programs) is recorded as a log
- Logs store details like timestamp, action, and user
- They are essential for monitoring and security ananlysis

* Why Logs Matter (SOC Use Cases)
- Incident Response --> Identify when, how an attack happened
- Threat hunting --> Search for suspicious or malicious behaviour
- Alerting & Triage --> Build detection rules and investigate alerts

* Window Log Storage
- Logs are stored as binary [.evtx] files in
   C:\Windows\System32\winevt\Logs
- Each file represents a specific log category, such as
  - Application Logs --> Apps like IIS, SQL Server
  - Security Logs --> Logic, processes, user activity

* Viewing Logs (Event Viewer)
  - Tool: Event Viewer (eventvwr)
  - Key components :
    - Log Sources --> Categories of logs
    - Log list --> Individual events
    - Event Details --> Full log content (XML/plain text)
    - Filters/Search --> Find specific events

* Key Log Fields
  - Event ID → Unique identifier (e.g., 4625 = failed login)
  - Date & Time → When event occurred (system time)
  - Keywords → Success/failure status
  - Details → Full event data

* Important Note
  - There are hundreds to thousands of Event IDs.
  - Not all events are logged by default.
  - SOC analysts focus on the most useful and high-signal logs.

## c. Security Log: Authentication
<img width="361" height="336" alt="image" src="https://github.com/user-attachments/assets/f0b04148-1d0c-4494-a8de-7f073a8418fa" />
Core Idea : 
  - As a SOC analyst, you don’t know what attack is coming, so you rely on high-value logs.
  - In Windows, the Security log is the most important by default.
  - Two critical Event IDs:
    - 4624 → Successful logon
    - 4625 → Failed logon

<img width="582" height="266" alt="image" src="https://github.com/user-attachments/assets/ecc965fe-c4c1-4921-96e8-c38df2a59535" />

* Key Fields in 4624
  - Focus on a few important fields (don’t try to read everything):

  - Logon Type → How login happened (e.g., 10 = RDP)
  - Logon ID → Unique session identifier
  - Username → Account used
  - Source IP / Hostname → Where login came from

--> These are enough for most L1/L2 SOC investigations.

* How They’re Used Together
  - 4625 spikes → Possible brute force attack
  - Followed by 4624 success → Attacker gained access
  - Match:
    - Same IP
    - Same username
    - Close timestamps
--> This chain = compromised account

* Practical Investigation Flow
  1.Look for many 4625 (failed logins) → identify attacker IP
  2.Check if a 4624 (successful login) follows
  3.Confirm:
    - Logon Type 10 → RDP login
    - Same IP + user
  4.Extract:
    - Attacker IP
    - Compromised user
    - Logon ID of successful session
* Key Takeaway
    - 4625 = attack attempt
    - 4624 = attack success
    - Combine both to detect brute force → compromise

## d.Security Log : User management
🔹 Core Idea
Attackers don’t just log in — they often:

Create new accounts
Reset passwords
Add users to admin groups

👉 This is how they maintain persistence and escalate privileges.

🔹 Key Event IDs to Know
👤 Account Creation / Changes
4720 → User created
4722 → User enabled
4738 → User modified
👉 ⚠️ Common for backdoor accounts

🔒 Account Removal / Disruption
4725 → User disabled
4726 → User deleted
👉 ⚠️ Attackers may disable security/admin accounts

🔑 Password Activity
4723 → User changed password
4724 → Password reset
👉 ⚠️ Used to take over accounts

🛡️ Privilege Escalation
4732 → Added to group
4733 → Removed from group
👉 ⚠️ Adding to Administrators = major red flag

🔹 Log Structure (Very Important)
Each event has 3 key parts:

Subject → Who performed the action
Includes Logon ID (link back to login event 4624)
Object → Target user (e.g., new account)
Details → What changed (group name, attributes, etc.)

🔹 How to Detect a Backdoor Account
🧠 Investigation Flow:
Start from suspicious 4624 RDP login
Note the Logon ID
Search for:
4720 → New user created
Check:
Who created it (Subject)
Does Logon ID match attacker session? ✅
Look for:
4732 → Added to privileged groups

🔹 Red Flags
New user created right after RDP login
Unknown username (e.g., svc_sysrestore)
Added to:
Administrators
Other privileged groups
Same Logon ID as suspicious login

🔹 Key Insight
👉 Logon ID = timeline glue

Connects:
Login (4624)
User creation (4720)
Privilege escalation (4732)

🔹 Expected Answers Logic (for your lab)
User created → Look for 4720 after RDP login
Groups added → Check 4732 events for that user
Logon ID match → Compare with previous task

<img width="853" height="213" alt="image" src="https://github.com/user-attachments/assets/60644710-0a48-4972-b556-43134965d91d" />

## e. Sysmon: Process Monitoring 
  -> Why Process Monitoring Matters
      - Authentication logs (4624/4625) tell you who is compromised
      - But NOT how the attack happened
--> Process logs answer:

  -> What file was executed?
  -> Where did it come from?
  -> What did it spawn?

🔹 Two Ways to Monitor Processes
  1. Event ID 4688 (Security Log)
Logs process creation
Includes:
  - Command line
  - Parent process
❌ Disabled by default
❌ Limited visibility

  2. Sysmon Event ID 1 (Best Option)
Advanced process monitoring tool from Microsoft
Provides:
  - Process path (image)
  - Command line
  - Parent process
  - File hash & signature
✅ Much more powerful than 4688
❌ Needs manual installation

📍 Location in Event Viewer:
Applications & Services Logs → Microsoft → Windows → Sysmon → Operational

🔹 Key Fields in Sysmon Event ID 1

🧠 Process Info
Image → Executable path
CommandLine → How it was run
Process ID (PID)

🌳 Parent Info
ParentImage → Who launched it
👉 Helps build attack chain

🔐 Binary Info
Hashes → Identify malware
Signature → Legit or suspicious

👤 User Context
User
Logon ID → Link back to login session

🔹 Investigation Flow (Very Important)
Start with compromised user (e.g., Sarah)
Look at Sysmon logs (Event ID 1)
Find:
-> Browser process (e.g., chrome.exe, firefox.exe)
* Check:
  - Files downloaded
  - Suspicious executables launched
* Correlate:
  - With other Sysmon events (like network connections)
* Trace:
Full attack chain (browser → download → execution)

🔹 What You’re Solving in This Lab
You’ll need to:

  ✅ 1. Identify Browser
Look for common processes:
chrome.exe
firefox.exe
msedge.exe
  ✅ 2. Find Downloaded File
Look at:
Browser → spawning another process
File path in Image / CommandLine
  ✅ 3. Find Download URL
Use other Sysmon events (likely Event ID 3 – Network connection)
Match:
Same process (PID or Image)
👉 This reveals the URL/domain

🔹 Key Insight
👉 Attack chain usually looks like:
Browser → Download → File executed → Malicious activity

🔹 Pro Tip
Always correlate using:
Process ID (PID)
Logon ID
👉 This is how SOC analysts reconstruct attacks accurately

## f. Sysmon : File and Network
Core Idea
Process creation (Event ID 1) shows what started the attack
Additional Sysmon events show what the malware did next

👉 You need multiple event IDs together to reconstruct the full attack chain.

🔹 Important Sysmon Event IDs
📁 File & Registry Changes
11 → File Create
13 → Registry Value Set
👉 Detect:
Malware dropping files
Persistence mechanisms (e.g., autorun registry keys)
🌐 Network Activity
3 → Network Connection
22 → DNS Query
👉 Detect:
Connections to attacker servers (C2)
Domain lookups for malicious infrastructure
🔹 Key Concept: Correlation via ProcessId
Many Sysmon events lack full context
They don’t always show:
Logon ID
Parent process

👉 Solution:

Use ProcessId (PID)
Match it with Event ID 1
→ Get full process details (who ran it, from where, how)
🔹 Investigation Flow (Real SOC Workflow)
🧠 Step-by-step:
Start with malicious process (Event ID 1)
Note:
ProcessId
Image (file path)
📁 3. Find Persistence
Search:
Event ID 11 (file created)
Event ID 13 (registry modified)
Same ProcessId
👉 Identify file or registry used for persistence
🌐 4. Find C2 Connection
Search:
Event ID 3 (network connection)
Same ProcessId
👉 Extract:
Destination IP + Port
🌍 5. Resolve Domain
Search:
Event ID 22 (DNS query)
Match:
Same process or timestamp
👉 Find domain linked to that IP
🔹 What You’re Solving in This Lab
✅ 1. Persistence File
Look for:
Event ID 11 or 13
Clues:
Suspicious path (e.g., AppData, Startup, Temp)
✅ 2. Command & Control (C2)
Event ID 3
Extract:
Destination IP + Port
✅ 3. Malicious Domain
Event ID 22
Match:
DNS query tied to the same process/IP
🔹 Key Insight

👉 Full attack chain:

Browser → Download → Malware executed (ID 1)
→ Persistence (ID 11/13)
→ C2 connection (ID 3)
→ DNS resolution (ID 22)
🔹 Pro Tips
Always correlate using:
ProcessId (PID)
Timestamp
Check common persistence locations:
AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
Suspicious outbound traffic = high priority

## g. PowerShell: Logging Commands

🔹 Core Problem
Tools like Sysmon (Event ID 1) only show:
powershell.exe was executed
❌ But NOT:
What commands were run inside it

👉 This creates a visibility gap for SOC analysts.

🔹 Why PowerShell is Dangerous
One process → many actions:
📂 Read files (Get-Content)
👤 Enumerate users (Get-LocalUser)
🌐 Download malware (Invoke-WebRequest)
📤 Data exfiltration

👉 All inside a single session, no new process logs

🔹 Solution: PowerShell History File

📍 Location:

C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
🔹 What It Records
Every command typed in PowerShell
Updated immediately after pressing Enter
Stored as plain text
🔹 Key Advantages
Shows actual attacker commands
Persists after reboot
Separate file for each user
🔹 Limitations
❌ No command output
❌ No script content (e.g., .ps1 files)
❌ Can be deleted by attacker
🔹 Investigation Steps (For Your Lab)
✅ 1. Find First Command
Open:
ConsoleHost_history.txt (Administrator)
The first line = first executed command
✅ 2. Find Execution Time
Right-click file → Properties
Check:
Created or Modified date
👉 That’s your timestamp
✅ 3. Find the Flag
Check:
Administrator history
Other users’ history files
👉 Look for something like:
THM{...}
🔹 Pro Tips
Attackers often:
Download malware → Invoke-WebRequest
Use IEX (Invoke-Expression)
Always check all user profiles, not just Administrator
🔹 Key Insight

👉 PowerShell history = attacker’s command timeline

# 2. Window Threat Detection 1
## a. Introduction 
Explore how threat actors access and breach Windows machines
Learn common Initial Access techniques via real-world examples
Practice detecting every technique using Windows event logs

## b. Initial Access
  - Initial Access is the first stage of a cyber attack where a threat actor successfully gains entry into a target system. Think of it as breaking through the “front door” of a system.
  - There are two main ways attackers achieve Initial Access:

  1. Exposed Services
--> Attackers exploit systems that are directly accessible from the internet, such as RDP, web servers, or mail servers. These systems are constantly scanned for weak passwords, misconfigurations, or vulnerabilities.

Common MITRE techniques:
  - T1133 – External Remote Services: Exploiting exposed services like RDP, SSH, or VNC.
  - T1190 – Exploit Public-Facing Application: Exploiting vulnerable applications like web or mail servers.
<img width="664" height="196" alt="image" src="https://github.com/user-attachments/assets/56a1be14-6e27-4576-a4b7-6188039d11cf" />

  2. User-Driven Attacks
  - Attackers trick users into helping them infect the system. This relies on human behavior rather than technical vulnerabilities.

Common MITRE techniques:
  - T1566 – Phishing: Users are tricked into opening malicious links or attachments.
  - T1091 – Removable Media: Infection through USB devices or external media.
<img width="666" height="191" alt="image" src="https://github.com/user-attachments/assets/54fad2ae-4f2a-49cb-a1a8-e353763bccc7" />

Key Idea
  - Attackers will use any available method—technical weaknesses or human mistakes—to gain their first foothold in a system

## c. Intial Access via RDP
🔴 Key Risks of Exposed RDP
    - RDP exposed to the internet is heavily targeted by botnets
    - Weak passwords (e.g., 12345678) are quickly brute-forced
    - Millions of RDP systems are publicly accessible and frequently compromised
    - Compromised RDP often leads directly to ransomware attacks (sometimes called “Ransomware Deployment Protocol”)

🧠 Attack Flow in the Scenario
  - Scanning phase
    - Attackers find open RDP ports (not directly visible in logs)
  - Brute-force attack
    - Many failed login attempts
    - Detected using:
      - Event ID 4625 (failed logins)
      - Logon Types 3 and 10
      - External IP addresses
  - Successful RDP login (Initial Access)
    - Detected using:
      - Event ID 4624 (successful login)
      - Logon Type 10
    - Identifies the account used for compromise
  - Post-exploitation activity
    - Attacker interacts via RDP
    - Use Logon ID correlation to link activities
    - Sysmon logs show processes created by attacker

🛠️ SOC Detection Strategy
    - Monitor spikes in 4625 failed logins
    - Identify brute-force patterns from single IPs
    - Track successful 4624 Logon Type 10 events
    - Correlate Logon ID across Security + Sysmon logs

## d. Initital Access via Phishing
This section explains how phishing is still one of the most dangerous attack methods because it bypasses firewalls by targeting users directly.

🔴 Key Phishing Techniques
1. Malicious Binary Attachments

Attackers send files like:

.exe
.com
.scr
.cpl

They often disguise them as:

invoice.pdf.exe
meeting.com

👉 Windows hides extensions by default, making them look harmless.

2. LNK Shortcut Attacks

Attackers use .lnk files (shortcuts) that:

Look like normal documents or websites
Actually execute PowerShell or scripts
Download malware like RATs (e.g., RemcosRAT)

Example behavior:

LNK file → runs PowerShell → downloads malware → executes it

🔍 Detection idea (SOC perspective)
Check file type mismatches
Inspect shortcut target (.lnk properties)
Look for PowerShell execution chains in logs
Monitor downloads from suspicious domains

## e.This section explains how SOC analysts detect malicious file downloads and execution chains using Sysmon logs in Windows.

🔍 Key Idea

Attackers often use phishing attachments like:

.exe (direct malware)
.zip, .rar (compressed malware)
Double extension files like invoice.pdf.exe

Sysmon helps track every step of the attack lifecycle.

⚙️ Typical Attack Chain (Sysmon Events)
1. Browser launch (Event ID 1)
A web browser (Edge, Chrome) is started
Parent process: explorer.exe

👉 Shows user activity begins

2. File download (Event ID 11)
Browser downloads file into Downloads folder

Example:

invoice.zip

👉 Indicates possible phishing attachment delivery

3. File extraction (Event ID 11)
Archive is unpacked using Explorer or tools like 7-Zip

Example:

invoice.pdf.exe

👉 This is often the malicious payload hidden behind a fake PDF

4. Execution (Event ID 1)
User runs the file
Parent process: explorer.exe
Child process: malicious .exe

👉 This is the actual compromise (execution stage)

⚠️ Important Insight: LNK Attacks
.lnk files (shortcuts) are dangerous because:
They may not show clear execution logs
They can directly trigger PowerShell or malware
How SOC detects LNK attacks:

Even if execution is hidden, you can still detect it by:

Checking file creation in Downloads
Finding .lnk files before execution
Observing suspicious parent process like:
explorer.exe → powershell.exe

👉 Key idea:

LNK execution itself is often invisible in logs, but file creation + PowerShell behavior reveals the attack.

## f. Initial Access via USB
This section explains how USB/removable media attacks are still a major Initial Access technique, even in modern cloud environments.

🔴 Why USB attacks are still dangerous
USB attacks bypass firewalls completely
Can work without internet connection
Can spread automatically or via user action
Still actively used by malware families like:
Raspberry Robin
Camaro Dragon
📦 Common USB attack scenarios
1. Fake “gift USB” attack
Victim receives infected USB (e.g., “HR gift”)
User plugs it in and opens a file
Malware executes silently in background
2. Supply chain / third-party infection
USB used at external services (e.g., print shop)
Their system is already infected
Malware spreads back to user’s USB and home PC
⚙️ Common USB malware techniques

Attackers often:

Hide real files and create .lnk shortcuts
Replace folders with fake .exe files

Use double extensions like:

photo.jpg.exe
Trick users into clicking via Explorer UI
🔍 How SOC detects USB-based attacks
Key indicators in Sysmon:
1. Execution from USB drive

Look for process paths like:

E:\malware.exe

👉 This shows execution from external media

2. Malware execution via Explorer

Parent process:

explorer.exe

Child process:

USB file (.exe / .lnk)
3. File creation on USB
New suspicious files appearing on removable drive
Example:
.lnk
.exe
hidden payload files
⚠️ Key SOC insight

USB attacks look very similar to phishing:

Both rely on user interaction
Both execute via Explorer (GUI)
Both are hard to trace without Sysmon correlation
