# 1. Definition of Log
- A "log" is a record of events, activities, or access attempts on a system. Logs are crucial for monitoring, detecting suspicious behavior, and investigating incidents. They track who logged in, when, from where, and what actions were performed.

# 2. Types of Logs

## Windows Event Logs

Windows Event Log is a service that records detailed information about important events occurring on a computer (from user activities and applications to the operating system). Understanding and effectively managing these logs is mandatory to ensure the security, stability, and performance of the entire IT infrastructure.

The system is organised into different channels. The five most important default log channels commonly used in security monitoring and investigation are:

| Log Name | Function | Example |
| :--- | :--- | :--- |
| **Application** | Records events from software and applications installed on the system, including errors, warnings, or information about their activities. | SQL Server database connection error, accounting software crash, or an alert from an antivirus program. |
| **Security** | The **most important channel** for security monitoring. Records security‑related events based on the system's Audit Policy, such as success/failure of login attempts, user permission changes, or other administrative activities. | - Successful/failed logins (Event IDs 4624, 4625). These are key indicators for detecting brute‑force attacks.<br>- New process creation (4688), which may indicate malware execution.<br>- New service installation (7045), often used for persistence after compromise.<br>- Security log clearing (1102), a common attacker action to erase traces. |
| **System** | Records events from the Windows operating system and system services. Information here typically relates to driver errors, hardware failures, or system service issues during startup and operation. | A system service fails to start, low disk space warning, or a device driver error. |
| **Setup** | Records events related to the installation and updating of programs, applications, and the operating system itself. | Information about a successful security patch installation or an error during a Windows upgrade. |
| **Forwarded Events** | A special log that stores events collected and forwarded from other computers on the network, allowing administrators to monitor the entire system centrally. | A collector server receives and stores security events from hundreds of workstations in the company. |

---

## Linux System Logs

On Linux, most log files are stored centrally in the `/var/log/` directory and are managed by logging systems such as `syslog` or `rsyslog`. These log files provide detailed information about the entire system's activity and are essential tools for troubleshooting and security monitoring.

Below are some of the most important log files that administrators and security professionals need to know:

| Log File Name | Function | Example |
| :--- | :--- | :--- |
| **/var/log/messages** (RHEL/CentOS) or **/var/log/syslog** (Ubuntu/Debian) | The system's general log file. Stores all system activity information, including kernel messages, service logs (mail, cron), and system processes. This is the first log file to check when an issue occurs. | Information about SSH service startup, a scheduled cron job run, or a system process notification. |
| **/var/log/auth.log** (Ubuntu/Debian) or **/var/log/secure** (RHEL/CentOS) | Contains **all authentication and authorisation information** for users. Helps track activities such as SSH logins (success/failure), `sudo` command usage, and password‑related changes. | A typical line in `/var/log/auth.log`: `May 3 18:23:56 host login[673]: pam_unix(login:session): session opened for root` indicates a successful root login session. |
| **/var/log/kern.log** | Records messages from the Linux kernel. This log file is very useful for troubleshooting hardware‑related errors, device driver issues, or memory and process problems at the kernel level. | Error messages about a USB driver failing to load, or a warning about a memory segmentation fault caused by an application. |
| **/var/log/audit/audit.log** | The log file of the Linux Auditing system (auditd). Records detailed security events based on rules defined by the administrator. auditd can monitor access to important files, system configuration changes, or the execution of specific commands. | An audit.log entry recording that a user (UID=500) executed the `cat` command to read the configuration file `/etc/ssh/sshd_config`. The log will contain details about the time, process, user, and action. |
| **/var/log/cron** | Contains information about scheduled tasks (cron jobs). This log records the start time, end time, and any errors that occur during the execution of cron tasks. | A message indicating that a daily backup script ran successfully or encountered an error. |

---

## Quick Comparison and Other Components

The table below summarises the key differences between the two logging systems:

| Feature | Windows Event Log | Linux System Log |
| :--- | :--- | :--- |
| **Management tools** | Event Viewer (GUI), `wevtutil` (CLI) | Plain text files, managed by `syslog`, `rsyslog`, or `systemd-journald`. |
| **Storage format** | Binary files (`.evtx`) | Plain text files (directly readable). |
| **Log retrieval** | Event Viewer or PowerShell commands. | Commands such as `cat`, `grep`, `tail`, `awk`, `journalctl` (on systemd‑based distributions). |
| **Default log types** | Application, Security, System, Setup, Forwarded Events. | `syslog`/`messages`, `auth.log`/`secure`, `kern.log`, `audit.log`, `cron`, `dmesg`, `boot.log`. |

In addition to the core log types above, both operating systems store logs from many other applications and services. For example, on Windows: logs from IIS web server, PowerShell, Firewall; on Linux: logs from web servers (Apache, Nginx), databases (MySQL, PostgreSQL), and email servers (Postfix). Logs for these applications typically have their own formats and locations.


# 3.From Raw Logs to Alerts in SOC
Detailed Process: From Raw Log to Alert in a SOC
--> The lifecycle of a security event starts with a raw log entry and ends with a response action. This process follows a standard processing flow, often called the "SIEM Log Flow".

| Phase | English Name | Main Task | Common Tools |
| :--- | :--- | :--- | :--- |
| 1 | Log Generation | Applications and systems record their activities. | Nginx, Apache, Windows Event Log, Syslog, AWS CloudTrail |
| 2 | Log Collection | Centralise logs from all sources to a SIEM. | Logstash, Fluentd, Filebeat, Syslog‑ng, Apache Kafka |
| 3 | Parsing | Split raw log strings into structured data fields. | Grok filter (Logstash), Parser (Fluentd), KQL, SPL |
| 4 | Normalisation | Standardise field names from different sources. | Mutate filter (Logstash), ECS (Elastic Common Schema), CIM (Splunk) |
| 5 | Enrichment | Add context (geo, blacklists, etc.) to data. | GeoIP filter, VirusTotal API, MISP, Logstash HTTP filter |
| 6 | Indexing & Storage | Store processed data and index for fast search. | Elasticsearch, Splunk Indexer, Azure Data Explorer |
| 7 | Correlation & Detection | Analyse and correlate events to detect threats. | Splunk ES, Microsoft Sentinel Analytics, Wazuh, Sumo Logic |
| 8 | Alerting | Generate alerts when detection conditions are met. | Kibana Alerting, ElastAlert2, Splunk Notable Events |
| 9 | SOC Analysis & Triage | Analyse and triage alerts to identify real threats. | TheHive, JIRA, ServiceNow, SA‑Investigator for Splunk |
| 10 | Response & Remediation | Contain, eradicate the threat, and restore systems. | SOAR (Cortex XSOAR, Phantom), EDR, Logic Apps, Shuffle |

## Step 1: Log Generation
* Definition: This is the starting point, where devices, servers, applications, and services within the IT infrastructure record their activities, events, errors, or actions. Each log line is a piece of the puzzle, a "narrative" of what is happening inside the digital infrastructure.

Typical Log Structure: Logs usually contain the following components:
  - Timestamp: When the event occurred.
  - Hostname / IP: The source that generated the log.
  - Service / Application: Name of the service or application.
  - Log Level / Severity: Severity level (INFO, WARN, ERROR, DEBUG).
  - Message / Payload: Detailed content of the event.
  - Metadata: Additional information (User ID, Session ID, Source/Destination IP, etc.).
  - Concrete Example: A web server records an access request. This is a raw Nginx log:
192.168.1.100 - - [15/May/2026:13:45:22 +0000] "POST /login.php HTTP/1.1" 401 785 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

* Common Tools: Logs are generated by the systems themselves. Common log‑generating systems include: Operating systems (Windows Event Log, Linux Syslog), Web servers (Nginx, Apache, IIS), Databases (MySQL, PostgreSQL, MongoDB), Custom applications, Network devices (Firewall, Router, Switch), Cloud platforms (AWS CloudTrail, Azure Monitor).

* Reason: Logs are the living evidence essential for understanding system behaviour. Detailed, well‑structured logging is the foundation for all monitoring, analysis, and security investigation activities.

## Step 2: Log Collection
* Definition: The process of securely, reliably, and near‑real‑time collecting raw logs from countless different sources (servers, network devices, applications) and bringing them to a single centralised system, typically a SIEM (Security Information and Event Management).

* Structure: This stage uses agents or forwarders installed on each server to read log files, or configures devices to send logs via standard protocols like Syslog. Collected data is then placed into a message queue for further processing.

* Example: Filebeat configuration (filebeat.yml) to read Nginx logs and send them to Logstash with additional metadata:

filebeat.inputs:

type: filestream
enabled: true
paths:

/var/log/nginx/access.log
fields:
service: "nginx-webserver"
env: "production"
output.logstash:
hosts: ["logstash.internal:5044"] 


* Common Tools: Logstash (part of ELK Stack, powerful with many plugins), Fluentd (lightweight, efficient, popular in Cloud‑Native and Kubernetes environments), Filebeat / Winlogbeat (lightweight, for forwarding log files or Windows Events), Syslog‑ng / Rsyslog (standard for network device logs), Kafka (central queue to buffer logs when data volume is high).

* Reason: Centralising logs solves the "distributed data" problem. Instead of having to SSH into hundreds of servers to view logs, an analyst can access a single place. This not only saves time but also creates a unified view, helping to detect attacks that spread laterally across the entire infrastructure.

## Step 3: Parsing

* Definition: Raw logs are often unstructured text strings, making them hard for computers to understand and process automatically. Parsing is the process of analysing this text string based on predefined patterns (e.g., RegEx, Grok patterns) to extract separate, well‑structured data fields.

* Structure: From a flat text string, parsing produces a JSON object or a table with data columns, each column having a specific meaning (e.g., timestamp, src_ip, http_status, username).

* Example: With the raw Nginx log:
192.168.1.100 - - [15/May/2026:13:45:22 +0000] "POST /login.php HTTP/1.1" 401 785

* After parsing, we get a JSON structure:

{
"client_ip": "192.168.1.100",
"timestamp": "15/May/2026:13:45:22 +0000",
"http_method": "POST",
"uri": "/login.php",
"http_version": "1.1",
"http_status": 401,
"bytes_sent": 785
}

* Common Tools: Logstash (using Grok filter, the most powerful parsing plugin, but can be resource‑intensive), Fluentd (using built‑in parsers, efficient with simple configuration), Kusto Query Language (KQL) (used in Microsoft Sentinel, parses at query time), Splunk Search Processing Language (SPL).

* Reason: Parsing is the critical step that turns "machine‑readable" data into "machine‑understandable" data. It allows computers to perform logical operations, comparisons, and statistics on specific data fields. This underpins every complex query and automated analysis that follows.

## Step 4: Normalization
* Definition: Different device vendors may use different field names and data formats to represent the same concept. For example, a destination host may be logged as dest_host, dst_host, target.host, or destination_hostname. Normalization is the process of mapping all these heterogeneous field names and formats into a single, unified common schema.

* Structure: A standard schema (e.g., Elastic Common Schema - ECS) defines a set of mandatory and recommended field names (e.g., source.ip, destination.port, user.name, event.action, event.outcome, event.severity).

* Example: Two logs from two different firewalls:
  - Firewall A: dst=10.0.0.5, action=BLOCK
  - Firewall B: dest_ip=10.0.0.5, outcome=deny

* After normalization, both are standardised into a common format, for example under the ECS schema:

{
"destination": { "ip": "10.0.0.5" },
"event": { "action": "block" }
}

* Common Tools: Logstash (using mutate and rename filters to change field names), Azure Monitor / Microsoft Sentinel (KQL parser functions), Splunk (aliases, field extraction, CIM – Common Information Model), Elastic Common Schema (ECS) is a de facto standard.

* Reason: The importance of normalization is to create a common language, enabling correlation of data from countless different sources. Without this step, writing a detection rule for an attack across the entire system would be impossible or extremely complex. It is the foundation of an effective SIEM.

## Step 5: Enrichment

* Definition: This step adds contextual information from external data sources to the already‑processed log. The goal is to "enrich" the log, providing deeper insight and helping the analyst understand the importance and impact of the event without having to manually look it up.

* Structure: Enriched data is usually added as new fields. Examples: geoip (geographic information), threat_intel (IP/domain blacklists), asset_criticality (criticality of the asset), user_identity (information about the user).

* Example: A log contains the field src_ip: "45.33.22.11". The enrichment step automatically looks up this IP:

  - GeoIP: Adds geo.country: "VN", geo.city: "Ho Chi Minh City".
  - Threat Intelligence: Checks and adds threat.intel.score: 85, threat.intel.category: "Malware CnC".
  - The enriched log becomes:
{
"src_ip": "45.33.22.11",
"geo": { "country": "VN", "city": "Ho Chi Minh City" },
"threat": { "intel_score": 85, "category": "Malware CnC" }
}

* Common Tools: Logstash (filters like geoip, translate, http), Azure Sentinel (using playbooks or watchlists), Splunk (lookups), Threat Intelligence sources: VirusTotal, AlienVault OTX, MISP.

* Reason: Enrichment reduces Mean Time to Respond (MTTR) and increases the accuracy of decisions. An alert from IP 45.33.22.11 would have a completely different priority if you know that the IP comes from the United States (lower risk) versus a country with high cyber‑attack density, or worse, if it appears on a blacklist of malware command‑and‑control (C2) servers.

Step 6: Indexing & Storage

* Definition: The process of storing processed log data in a specialised database capable of ultra‑fast search and querying. Indexing is like creating a table of contents for a thick book – it allows the search engine to instantly locate any keyword or field value without having to read all the data.

* Structure: Data is stored as inverted indexes in distributed databases specifically designed for search and analytics.

* Example: The JSON log data after the previous steps is sent to Elasticsearch. Elasticsearch analyses the JSON structure, indexes every field, and stores them in shards on disk, ready for future queries.

* Common Tools: Elasticsearch (the heart of the ELK Stack, widely popular), Splunk Indexer (stores data as already‑analysed events), Azure Data Explorer (for Microsoft Sentinel), AWS OpenSearch (open‑source version of Elasticsearch), ClickHouse (for high‑performance log analytics solutions).

* Reason: Without this step, every query would be unbearably slow and completely infeasible in an environment with terabytes of logs per day. Indexing is the key that turns a raw data lake into an interactive, real‑time analytics tool.

## Step 7: Correlation & Detection

* Definition: This is the heart of the SIEM. Correlation is the process of analysing and combining individual events from many different sources over time to identify complex patterns that may indicate an ongoing attack. Detection is applying logic or techniques (e.g., rule‑based, threshold‑based, anomaly detection) to the correlated data to trigger an alert.

* Structure: Rules can be built in various types depending on the tool, for example:

* Match Rule: Triggers when a single record satisfies a condition.
  - Threshold Rule: Triggers when the number of events exceeds a threshold within a time window.
  - Chain Rule: Triggers when a sequence of events occurs in a specific order.

* Example: A brute‑force attack detection scenario. The rule could be built as follows:

  - Step 1 (Correlation): Search for failed login events (normalizedAction = 'logon' AND success = false) coming from the same source IP address (source.ip) targeting different user accounts (user.name).
  - Step 2 (Detection): If there are more than 10 such events within 5 minutes → Trigger alert "Possible Brute‑Force Attack from IP X".

* Common Tools: Most SIEM platforms have powerful correlation engines: Splunk Enterprise Security (Correlation Search), Microsoft Sentinel (Analytics Rules), Elastic Stack (using Watcher/Kibana Alerting), Sumo Logic Cloud SIEM (various rule types), Wazuh (Ruleset).

* Reason: A single isolated event (e.g., one failed login) is usually harmless. Only when many events are combined does the full picture of an attack emerge. Correlation helps eliminate false positives, detect complex attack campaigns, and is the soul of every modern intrusion detection system.

## Step 8: Alerting

* Definition: When the rules in the previous step are successfully triggered, an alert is generated. An alert is a structured record containing detailed information about the detected threat, including severity, affected assets, and timestamps. The alert is placed in a queue for SOC analysts to process.

* Structure: An alert typically contains: Alert name, Description, Severity (Critical, High, Medium, Low), Relevant IP addresses/Hostnames, Timestamp, and important contextual fields.

* Example: In Splunk, a correlation search generates a notable event (a type of alert) with detailed information such as impact level, risk score, and recommended response actions.

* Common Tools: This function is a core part of SIEM/Alerting platforms:
  - Kibana Alerting / Watcher (for Elastic Stack)
  - Splunk Enterprise Security Notable Events
  - Microsoft Sentinel Analytics Rules
  - ElastAlert2 (standalone tool for Elasticsearch, very flexible)

* Reason: Alerting is the mechanism for the system to automatically "speak up" when it detects an anomaly, instead of requiring humans to constantly watch. It reduces the Mean Time to Detect (MTTD) from hours or days down to seconds or minutes.

## Step 9: SOC Analysis & Triage

* Definition: SOC analysts (usually L1 level) start the analysis and triage process. They examine the generated alerts, validate their accuracy (true positive vs. false positive), assess the severity and scope of impact, and gather evidence for the next step.

* Structure: The analysis process typically follows these steps:
  - Prioritization: Determine priority based on severity and criticality of the affected asset.
  - Investigation: Use SIEM/EDR to query deeper, looking for other related events based on Who (User), What (Process), Where (IP/Hostname), When (Time).
  - Validation: Check indicators (IOCs) against threat intelligence sources.
  - Decision: Classify as True Positive (real threat) that needs escalation, or False Positive (false alarm) that can be closed.

* Example: An alert "High number of failed logins from IP X" is raised. The L1 analyst will:
  - Determine if IP X is an internal IP (if internal → might be a compromised machine).
  - Look up IP X on threat intelligence sources (e.g., VirusTotal) to see if it is blacklisted.
  - Query the SIEM to see what other behaviours IP X has performed (port scans, connections to strange addresses, etc.).
  - If many suspicious signs are found → conclude True Positive and create a ticket in the ITSM system to escalate to L2/L3 for deeper investigation. If not → conclude False Positive (e.g., due to an internal scanning application) and close the alert.

* Common Tools: These tools are often integrated into the SIEM ecosystem:
  - Splunk Investigation Workbench / SA-Investigator (allows investigation per entity: asset, identity)
  - Microsoft Sentinel Incidents & Investigation Graph
  - TheHive (open‑source investigation and case response management)
  - JIRA / ServiceNow for ticket management

* Reason: Not every generated alert is a real threat. Without analysis and triage, the SOC team would be overwhelmed by thousands of false positives each day, leading to missing real threats (alert fatigue). Triage helps filter out noise and focus resources on incidents that are highly likely to be genuine attacks.

## Step 10: Response & Remediation

* Definition: This is the final and most important step in the incident management lifecycle. After a threat is confirmed as a True Positive, response actions are taken to contain, eradicate the malicious actor, restore the system to a safe state, and prevent recurrence.

* Structure: The response process includes four main phases:

  - Containment: Isolate the affected system from the internal network, block the attacking IP on the firewall, disable compromised user accounts.
  - Eradication: Scan and remove malware, close backdoors, remediate the security vulnerabilities that were exploited.
  - Recovery: Restore data from clean backups, bring the system back to normal operation.
  - Post‑Incident Activities: Root Cause Analysis (RCA), update detection rules, improve security processes.

* Example: After confirming a compromised server, the SOC analyst (L3) triggers a SOAR playbook to:
  - Automatically isolate the server from the internal network.
  - Collect all logs and a VM snapshot for investigation.
  - Create a ticket for the infrastructure team to check the vulnerability.
  - If needed, block the attacker’s IP on the perimeter firewall.

* Common Tools: Response is strongly automated by SOAR (Security Orchestration, Automation, and Response) platforms and supported by ITSM tools:
  - SOAR Platforms: Cortex XSOAR, Splunk Phantom, Microsoft Sentinel Logic Apps (SOAR), Shuffle.
  - ITSM Tools: Jira Service Management, ServiceNow.
  - EDR Tools: CrowdStrike Falcon, Microsoft Defender for Endpoint, Carbon Black (allow remote endpoint isolation).

* Reason: The ultimate goal of a SOC is not only to detect attacks but to contain and remediate them as quickly as possible. Automation of responses through SOAR significantly reduces Mean Time to Respond (MTTR), frees human resources from repetitive tasks, and ensures actions are executed consistently and accurately.

# 4. Examples of malware:
## 1. Trojan Emotet – spread via macro
Log alert:
Alert: PowerShell spawns from WinWord.exe, downloads file from suspicious domain, executes encoded command.

3 constraints:
  - Parent process is winword.exe (Microsoft Word) spawning powershell.exe.
  - Network connection to a low-reputation domain (newly registered, not in top 1M Alexa).
  - PowerShell command line contains an abnormally long base64 string (encoded command) and the -EncodedCommand parameter.

## 2. Ransomware WannaCry – spreads via SMB
Log alert:
Alert: Multiple attempts to write encrypted files with .WNCRY extension, modification of boot record, and scanning port 445.

3 constraints:
  - Numerous files with extensions .WNCRY or .WNCRYT appear across multiple folders within seconds.
  - Process tasksche.exe or mssecsvc.exe attempts to overwrite the Master Boot Record (MBR).
  - Sends specially crafted SMB packets (EternalBlue) to multiple internal IPs in sequence.

## 3. Keylogger – records keystrokes
Log alert:
Alert: Process injects code into explorer.exe, hooks keyboard interrupts, and writes log file to hidden system folder.

3 constraints:
  - Unsigned or invalidly signed process calls SetWindowsHookEx for global keyboard monitoring.
  - Injects into explorer.exe memory space using WriteProcessMemory + CreateRemoteThread.
  - Continuously writes a file containing keystroke data to a hidden path (e.g., C:\ProgramData\logs\kb.log).

## 4. Backdoor Cobalt Strike – beaconing
Log alert:
Alert: Scheduled task 'MicrosoftEdgeUpdate' runs encoded PowerShell every 60 seconds, connecting to port 443 non-HTTP.

3 constraints:
  - Recurring task with short interval (<5 minutes), name mimics legitimate Windows tasks but not created by user.
  - TLS connection to a suspicious IP, but traffic is not pure HTTPS (unusual cipher, abnormal handshake sequence).
  - PowerShell runs with -WindowStyle Hidden -NoProfile -EncodedCommand, decoding to a command that fetches payload from C2.

## 5. CoinMiner – high CPU usage
Log alert:
Alert: Unknown executable with random name (e.g., svchost.exe but not in System32) uses 100% CPU, connects to mining pool.

3 constraints:
  - Process name resembles a system process but path is different (e.g., C:\Temp\svchost.exe).
  - Network connection to common mining pool ports (stratum+tcp://, ports 3333, 4444, 5555).
  - Executable has no digital signature or conflicting signature; SHA256 matches a known miner database entry.

## 6. Downloader – fetches additional payload
Log alert:
Alert: Script from HTA file launches mshta.exe, downloads binary from URL shortener, writes to AppData\Roaming.

3 constraints:
  - .hta file opened from email or Downloads folder calls mshta.exe with javascript: or vbscript: parameter.
  - Download URL uses a shortening service (tinyurl, bit.ly) or direct IP without domain name.
  - Downloaded file has .exe extension but is renamed to .tmp or .dat, then renames itself and executes.

## 7. Rootkit – hides processes
Log alert:
Alert: Driver loaded with suspicious name, hook on SSDT for NtQueryDirectoryFile, and no digital signature.

3 constraints:
  - Driver (.sys) loaded from an unprotected location (e.g., C:\Windows\Temp) with a name similar to a legitimate driver but altered slightly.
  - Calls ZwSetSystemInformation to load driver without going through Service Manager.
  - Hook detected on SSDT or IDT (using tools like GMER) that hides processes, files, or registry keys.

## 8. Spyware – screen capture
Log alert:
Alert: Process captures screen every second using BitBlt, sends image data to remote server via HTTP POST.

3 constraints:
  - Calls CreateCompatibleDC, BitBlt, GetDIBits at very high frequency (>1 per second).
  - Image data is compressed and lightly encoded (base64) then sent via HTTP POST to a domain unrelated to the organization.
  - Process runs under a legitimate name like svchost.exe or explorer.exe but from a user-writable directory.

## 9. Exploit EternalBlue – lateral movement
Log alert:
Alert: SMBv1 negotiation with malformed TRANS2 packet, followed by kernel shellcode execution.

3 constraints:
  - SMB packet with TRANS2 structure containing abnormal data length (>= 4096 bytes) that holds shellcode.
  - Immediately after, lsass.exe or svchost.exe executes code from a memory region with PAGE_EXECUTE_READWRITE permission.
  - Connects to port 445 of multiple IPs within the same subnet, sequentially in ascending order.

## 10. Web Shell – backdoor on web server
Log alert:
Alert: HTTP request to .aspx/.php with long query string containing system commands, response contains cmd.exe output.

3 constraints:
  - Target file has a suspicious name (cmd.aspx, shell.php, uploader.asp) located in a web-writable directory.
  - Query string contains system commands such as whoami, net user, powershell URL-encoded.
  - User-Agent header is abnormal (e.g., Microsoft Office Protocol Discovery) or does not match normal traffic patterns.


# 5. WHere, How, Which, Why?

## 1. Where does this log appear?

Specifically, this log can appear in multiple locations within an enterprise system:

| Location | Path / Interface |
|----------|------------------|
| **SIEM** (e.g., Splunk, Microsoft Sentinel, QRadar) | Alerts & Incidents dashboard, or search with `index=windows sourcetype="WinEventLog:Security" event_id=4688`. |
| **EDR** (e.g., CrowdStrike, Defender for Endpoint, SentinelOne) | Process timeline, Detection Details showing MITRE ATT&CK `T1059.001` (PowerShell). |
| **Windows Event Log** | `Microsoft-Windows-PowerShell/Operational` (event IDs 4103, 4104) + `Security` (event ID 4688 – process creation). |
| **Sysmon** (if installed) | Event ID 1 (process creation), 3 (network connection), 11 (file create). |
| **Network logs** (Firewall/Proxy) | Proxy logs (Squid, Zscaler) or firewall logs (Palo Alto) showing connection to `suspected-domain.xyz`. |

> In a real environment, these logs are typically aggregated into a SIEM from multiple sources to form a single alert.

---

## 2. How is it detected? (Detection mechanism)

The system does not rely on a single rule but combines **3 detection layers**:

### Layer 1: Endpoint (Sysmon + Windows Event Log)

- **Rule 1 (parent‑child process):**  
  Sysmon event ID 1 records `winword.exe` spawning `powershell.exe`.  
  This is abnormal because Word normally does not need to call PowerShell.

- **Rule 2 (long encoded command):**  
  PowerShell event ID 4104 (ScriptBlock Logging) records the command line.  
  The system detects the `-EncodedCommand` parameter with a base64 string longer than 800 characters – a sign of a malware download script.

### Layer 2: Network (DNS / Proxy / NDR)

- **Rule 3 (connection to malicious domain):**  
  When PowerShell executes `(New-Object Net.WebClient).DownloadString()`, it sends a DNS query.  
  DNS or proxy logs show that the domain `suspected-domain.xyz` is present in a threat intelligence feed (VirusTotal, ThreatFox, AbuseIPDB).

### Layer 3: Correlation engine (SIEM or EDR)

- This engine groups the 3 rules together within a short time window (e.g., 10 seconds).  
- If all 3 are true → a **High** severity alert is generated.

---

## 3. Which tools perform the detection?

| Tool | Role in detecting the log above |
|------|--------------------------------|
| **Sysmon** (with logging for process, network, file) | Records detailed process chain (event ID 1), network (event ID 3), created file (event ID 11). |
| **Windows PowerShell ScriptBlock Logging** (GPO: Turn on PowerShell Script Block Logging) | Records the entire script content, including decoded encoded commands. |
| **Windows Defender for Endpoint / Microsoft 365 Defender** | Automatically analyzes the chain, compares with cloud intelligence, generates alert. |
| **CrowdStrike Falcon / SentinelOne / Carbon Black** | EDR sensors on endpoints detect abnormal behavior. |
| **SIEM** (Splunk ES, Azure Sentinel, IBM QRadar) | Aggregates logs from multiple sources, runs correlation rule: `(process_parent = winword AND process_child = powershell) AND (command_line contains EncodedCommand) AND (network_dest in threat_intel)`. |
| **Threat Intelligence Platform** (MISP, ThreatFox, VirusTotal Enterprise) | Provides a list of malicious IPs/domains for matching. |

---

## 4. Asking "Why?"

(Why are these 3 rules indicators of Emotet malware?)

### Question 1: Why is `winword.exe` calling `powershell.exe` suspicious?

**Answer:** Word can indeed use PowerShell for automation (VBA + Shell), but in reality, most users never see this. Emotet abuses macros in `.doc` files to invoke PowerShell because PowerShell allows downloading and executing code without writing an `.exe` file to disk (fileless). This is a "Living off the Land" technique.

### Question 2: Why use a long `-EncodedCommand`?

**Answer:** `EncodedCommand` helps malware hide the command content from plain sight (only a base64 string is visible). A length >800 characters indicates a complex script – e.g., downloading additional payload, bypassing AMSI, establishing persistence. Legitimate scripts are usually short and do not use long encoded commands in the command line.

### Question 3: Why connect to a domain with a bad reputation?

**Answer:** Emotet needs to download stage 2 from a C2 server. That domain is often newly created (<30 days old), has no legitimate web content, and has already been recorded in threat intelligence feeds by sandboxes or the security community. Connecting to such a domain is typical behavior of a downloader trojan.

### Overall question: Why is any single rule insufficient to conclude malware?

**Answer:**  
- **Rule 1** (`winword → powershell`) could be a user intentionally running a legitimate macro (though rare).  
- **Rule 2** (long encoded command) could also be used for legitimate administrative purposes.  
- **Rule 3** (bad connection) could be a false positive if the domain was mislabeled.  

**All three occurring within seconds** raises confidence to nearly 100%, because it is very unlikely that a legitimate task accidentally satisfies all three constraints.

---

## Summary detection playbook

1. **Sysmon** records `winword.exe` creating `powershell.exe` → forwards to SIEM.  
2. **PowerShell logging** records command line containing a long `-EncodedCommand`.  
3. **DNS/Proxy logs** record a query to a suspicious domain.  
4. **SIEM correlation rule** triggers after 10 seconds if all 3 conditions are met.  
5. **EDR** automatically isolates the machine or blocks the process if automated response is configured.  

**Result:** Alert is raised, and the analyst sees the log as above and confirms **Emotet infection**.
