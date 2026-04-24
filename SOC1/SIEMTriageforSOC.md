# 1. Log Analysis with SIEM
## a. Introduciton
## b. Benefits of SIEM for Analysis
<img width="606" height="459" alt="image" src="https://github.com/user-attachments/assets/0cd26706-455a-4848-9c1e-6aadb105c78a" />
SIEM (Security Information and Event Management) is essential in a SOC because it helps analysts detect and investigate threats efficiently.

📌 1. Centralisation


Collects logs from multiple sources (network devices, cloud, endpoints, IDS, etc.)


Stores everything in one platform


Analysts don’t need to switch between systems


👉 Makes investigations faster and easier



🔗 2. Correlation


Links separate events into one meaningful story


Example:


IDS detects network scanning


SIEM correlates it with Windows logs to identify the device and user




👉 Helps determine if activity is malicious or normal



⏳ 3. Historical Events


Allows analysts to review past logs


Helps identify patterns or previous suspicious behavior


Example:


Checking if a login from an unusual location happened before





🧠 Key takeaway:
SIEM helps SOC analysts:


Centralize data


Correlate events


Analyze historical activity to detect threats faster

## c. Log Sources Overview
In a SIEM, logs come from different parts of an organisation and help analysts investigate security events.

🖥️ 1. Host-Based Log Sources
Comes from individual devices like:
Workstations
Servers (web, DNS, database, etc.)
Provides detailed activity on endpoints
👉 Very important because most attacks involve hosts

🌐 2. Network-Based Log Sources
Comes from network devices like:
Firewalls
Routers
IDS/IPS systems
Shows how devices communicate over the network
👉 Helps trace traffic and detect suspicious connections

🌍 3. Web-Based Log Sources
Comes from web applications
Tracks user activity and requests
Important for detecting web attacks (e.g., SQL injection, XSS)
👉 Often used as an entry point for attackers

☁️ Other Sources (mentioned briefly)
Cloud platforms (AWS, Azure)
Identity providers (Entra ID)
Third-party applications

⏰ Important SIEM Challenges
Time Pitfalls
Logs may use different time zones (UTC, local time, etc.)
SIEM may normalize time differently
👉 Always check time alignment during investigations

Log Normalisation
Logs come in different formats (JSON, XML, text)
SIEM converts them into a standard format
👉 Makes searching and correlation easier
🧠 Key takeaway:

SIEM collects logs from:
  Hosts (devices)
  Network systems
  Web applications

Then solves problems like:
  Time differences
  Different log formats
👉 So analysts can investigate everything in one consistent view

## d. Windows Logs
<img width="919" height="625" alt="image" src="https://github.com/user-attachments/assets/168a7d7f-2148-4e9a-a797-bbad4c71568c" />
In SIEM, Windows logs mainly come from two sources: Sysmon and Windows Event Logs (WinEventLogs). Together, they give strong visibility into system activity.

🧠 1. Sysmon (System Monitor)
Advanced logging tool installed separately
Provides deep visibility into system behavior
What it detects:
Process execution (malicious programs, PowerShell, etc.)
Network connections
File creation/modification
Registry changes
Process injection
Key Event Codes:
EventCode 1 → Process creation
EventCode 3 → Network connections

👉 Example use:

Detect powershell.exe with EncodedCommand → likely malicious execution
Detect unknown process connecting to IP/port → possible C2 traffic

🔐 2. Windows Event Logs (WinEventLogs)
Built-in Windows logging system
Contains 200+ log channels (Security, System, Application, etc.)
🔎 Security Logs (most important)

Used to detect:

Login attempts (success/failure)
User creation/modification
File and registry access
Process execution
Policy changes

👉 Key Event Codes:

4720 → User account created
4722 → User account enabled

✔ Used to detect persistence (e.g., attacker creating admin user)

⚙️ System Logs
Used to monitor:
  Services start/stop
  Service creation (persistence/privilege escalation)
  System errors and OS-level events

👉 Key Event Codes:
  7045 → New service created
  7036 → Service started/stopped

✔ Example attack:
Malicious service created → runs malware as SYSTEM → privilege escalation

🧠 Key takeaway:
  Sysmon = deep attack visibility (process, network, file activity)
  WinEventLogs = system + security actions (logins, users, services)
  Together → full attack timeline (from execution → persistence → network activity)
