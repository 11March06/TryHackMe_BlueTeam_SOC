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

# 5. ELK Stack 
<img width="1679" height="313" alt="image" src="https://github.com/user-attachments/assets/581c822f-8546-4905-b6fb-585e5cb0e7d8" />
<img width="1667" height="1009" alt="image" src="https://github.com/user-attachments/assets/c9e11b76-3912-4bd2-ab08-1b6069fba547" />
Check IP
<img width="1052" height="628" alt="image" src="https://github.com/user-attachments/assets/c4764abf-fa8d-4cc0-910c-7b5dedc8edf6" />
<IP> : 5601
--> Elastic
<img width="1204" height="1037" alt="image" src="https://github.com/user-attachments/assets/b29681b9-1625-4862-bff7-55587a00db4b" />

On Client-side
<img width="1227" height="900" alt="image" src="https://github.com/user-attachments/assets/7d11764a-43ae-4352-99da-15ffff6f3b90" />
Filebeat Connection - Successful 
<img width="651" height="492" alt="image" src="https://github.com/user-attachments/assets/8799ffc0-9c1d-412e-a710-c0236651dc92" />
Configuring Filebeat with INPUT and OUTPUT(IP:9200)
--> <img width="1301" height="848" alt="image" src="https://github.com/user-attachments/assets/519ad192-0f2c-4ce7-8b13-eb9c67d65e65" />


## Types of Logs - ELK STACK
## Ubuntu-Linux - with filebeat
auth.log
<img width="1208" height="777" alt="image" src="https://github.com/user-attachments/assets/74b64ac0-871b-4230-b354-eabfd2dc7522" />
auth.log → Records authentication events (logins, logouts, password changes). Useful for detecting failed logins, brute-force attacks, or unusual access attempts.


access.log
<img width="1367" height="743" alt="image" src="https://github.com/user-attachments/assets/86ef8152-c2a4-422e-98a8-aef64963d061" />
access.log → Web server log of client requests (IP, URL, status code, user-agent). Useful for traffic analysis, detecting web attacks, and performance monitoring.


audit.log
<img width="1325" height="916" alt="image" src="https://github.com/user-attachments/assets/1122ef5d-320b-4d63-96fa-1cd59130942f" />
audit.log → Tracks security auditing events (who did what, when, and where). Essential for compliance, forensic analysis, and monitoring user actions.


kern.log
<img width="1328" height="856" alt="image" src="https://github.com/user-attachments/assets/173dd67a-217f-4109-82bd-5159870d203d" />
kern.log → Contains kernel messages (hardware detection, driver issues, system crashes). Important for debugging hardware and OS-level problems.


syslog
<img width="1334" height="852" alt="image" src="https://github.com/user-attachments/assets/6a8c8d8b-706f-493b-ae04-d6b9c77388d1" />
syslog → General system log that aggregates messages from various services and daemons. Provides a broad view of system activity.


| **Log Type** | **Main Purpose**             | **Highlights**                          | **Key Difference**                  |
|--------------|-------------------------------|------------------------------------------|--------------------------------------|
| [auth.log](ca://s?q=auth.log_meaning)   | Authentication tracking        | Login success/failure, SSH attempts   | Focused on user access security      |
| [kern.log](ca://s?q=kern.log_meaning)   | Kernel & hardware events       | Driver errors, crashes, boot messages | Focused on system internals          |
| [syslog](ca://s?q=syslog_meaning)       | General system activity        | Daemon/service logs, cron jobs        | Broad coverage of system events       |
| [audit.log](ca://s?q=audit.log_meaning) | Security auditing              | User actions, compliance records      | Ensures accountability & compliance   |
| [access.log](ca://s?q=access.log_meaning) | Web traffic monitoring        | IPs, URLs, status codes               | Focused on web/application layer      |



# 6. Splunk
Download Splunk Enterprise (Server) in Ubuntu through website or wget.... Then, connecting through http://localhost:8000
<img width="1254" height="1562" alt="image" src="https://github.com/user-attachments/assets/af111dda-24b7-41e0-ac88-dadf77fc10b4" />
Settings --> Forwarding and receiving --> Add new receiving port : 9997 --> SAVE

Download Splunk Universal Forwarder (Client) in Ubuntu through website or wget..
<img width="656" height="437" alt="image" src="https://github.com/user-attachments/assets/a2df89e7-b4c3-409f-9527-482f6363c488" />

Configuring - monitoring these logs : auth, audit, access, kern, syslog
<img width="646" height="402" alt="image" src="https://github.com/user-attachments/assets/794e99cc-05fd-435b-b3e2-7106f6a4a959" />

Connection with 192.168.114.129:9997 
<img width="661" height="161" alt="image" src="https://github.com/user-attachments/assets/45e9a550-0001-4d8b-8feb-1128085fdc61" />

Testing logs 
<img width="650" height="293" alt="image" src="https://github.com/user-attachments/assets/1a02cf59-0d48-41b7-a112-2164006cc85e" />

Finally 
<img width="1577" height="1455" alt="image" src="https://github.com/user-attachments/assets/8cdca806-7a02-4bf3-8d5a-0d30fe160339" />
--> Seeing the log on dashboard, with five mentioned types of logs.







# Logs
## Auth logs
<img width="630" height="217" alt="image" src="https://github.com/user-attachments/assets/355698d4-49a3-405f-8ce8-4441ac913ec7" />

## Kern logs
<img width="656" height="71" alt="image" src="https://github.com/user-attachments/assets/be9de65e-c0a8-4873-9543-25863f615bcd" />

## Access logs 
Connecting http://localhost --> making log

## Syslog 
<img width="647" height="22" alt="image" src="https://github.com/user-attachments/assets/36bfdfaa-e1ff-491e-8249-78416e91e1df" />

## Audit log
<img width="643" height="66" alt="image" src="https://github.com/user-attachments/assets/b7959347-4ad5-4cb9-97ad-8e5e65971323" />

---
1. Syslog
General system log that records events from services, daemons, and the operating system.
--> Useful for monitoring overall system health and troubleshooting service issues.

2. Auth log
Authentication log that captures login attempts, sudo usage, and password failures.
--> Critical for detecting unauthorized access attempts or brute-force attacks.

3. Kernel log
Records messages from the Linux kernel, such as hardware events, driver issues, or module loading errors.
--> Helps diagnose hardware problems or kernel-level failures.

4. Apache access log
Tracks every HTTP request to the Apache web server: client IP, requested URL, status codes (200, 404, 500).
--> Useful for analyzing traffic, user behavior, and spotting suspicious activity.

5. Apache error log
Records server-side errors, misconfigurations, and application issues.
--> Essential for debugging web applications and identifying potential vulnerabilities.

6. Audit log
Security-focused log maintained by auditd. It records sensitive actions like file access, permission changes, or root-level commands.
--> Important for compliance, forensic analysis, and detecting abnormal behavior.
---

Client - side
## With Window
ELK 
Download and Setup Winlogbeat on PơerShell
<img width="980" height="514" alt="image" src="https://github.com/user-attachments/assets/042e31ab-8c23-4967-a3a1-2be61e5a184f" />
* winlogbeat.yml
<img width="467" height="370" alt="image" src="https://github.com/user-attachments/assets/49e6cba6-53ec-489b-b12d-5d6f25dcaa42" />

Connecting to 192.168.114.129:9200 (ElasticSearch)
On web 192.168.114.129 (Kibana)

--> Elastic View - With winlogbeat
<img width="1216" height="770" alt="image" src="https://github.com/user-attachments/assets/018a8aec-1047-4c1a-8999-9df6d67aa7cb" />

Making logs
1, Application log
<img width="1575" height="137" alt="image" src="https://github.com/user-attachments/assets/7230f4fc-68af-4082-9de5-63d124a5978d" />

2, System log
<img width="1562" height="51" alt="image" src="https://github.com/user-attachments/assets/92119475-9265-44af-83f6-0b7ab12aafda" />

3, Security log
<img width="799" height="346" alt="image" src="https://github.com/user-attachments/assets/fd044fc1-b123-4308-8fc3-f3aac9fa5129" />



Splunk
Check Server Side Connection 
<img width="684" height="38" alt="image" src="https://github.com/user-attachments/assets/76b8eff2-7935-4176-a550-9eb93c2d621b" /> Already done

Setup Universal Forwarder
<img width="498" height="382" alt="image" src="https://github.com/user-attachments/assets/3e4b1045-c18c-42c0-8e34-4afc978d4bc9" />

Configuring 'inputs.conf' with path: "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
<img width="752" height="515" alt="image" src="https://github.com/user-attachments/assets/0e3b875b-ceb9-481d-97c0-45af700fd853" />
Finally 
<img width="1655" height="1643" alt="image" src="https://github.com/user-attachments/assets/cf5db812-d88a-40aa-a695-346efc656dc8" />


# DIFFERENCE
## 🪟 Windows Event Logs (Application, Security, System)

| Log Type | Key Characteristics | Difference from Other Log Types | Concrete Example (from Kibana) |
|----------|---------------------|--------------------------------|--------------------------------|
| **Application** | Records events from software applications (errors, warnings, information). | – Does **not** contain security events (unlike `Security`).<br>– Does **not** contain OS/driver events (unlike `System`).<br>– Focuses solely on third‑party software and user‑installed applications. | `"message": "The storage optimiser couldn't complete boot optimisation on (C:) because: The user cancelled the operation. (0x89000006)"`<br>`"winlog.provider_name": "Microsoft-Windows-Defrag"` |
| **Security** | Most important for security: logon (success/failure), privilege changes, process creation, log clearing. | – Unlike `Application` (no application events).<br>– Unlike `System` (no driver/service events).<br>– Based on Audit Policy, not default app/system logging. | `"event.code": 4625`<br>`"message": "An account failed to log on. Account Name: Administrator, Source Network Address: 192.168.1.100, Failure Reason: Unknown user name or bad password."` |
| **System** | Events from the OS, drivers, system services (startup, driver errors, configuration changes). | – Unlike `Application` (no application events).<br>– Unlike `Security` (no authentication/authorization events).<br>– Focuses on system stability and hardware. | `"event.code": 16`<br>`"message": "The access history in hive \??\C:\ProgramData\Microsoft\Provisioning\Microsoft-Desktop-Provisioning-Sequence.dat was cleared updating 0 keys and creating 0 modified pages."`<br>`"winlog.provider_name": "Microsoft-Windows-Kernel-General"` |

---

## 🐧 Linux System Logs (syslog, auth.log, kern.log, audit.log – cron excluded)

| Log File | Key Characteristics | Difference from Other Log Types | Concrete Example (from lab) |
|----------|---------------------|--------------------------------|-----------------------------|
| **/var/log/syslog** | General system log: kernel, cron, mail, daemons, user processes. | – All‑purpose but less detailed than specialised logs.<br>– Unlike `auth.log` (only authentication).<br>– Unlike `kern.log` (only kernel).<br>– Unlike `audit.log` (requires installation, much more detailed). | `May 17 20:51:01 ubuntu1103 CRON[2345]: (root) CMD (test cron job)`<br>`May 17 20:51:02 ubuntu1103 systemd[1]: Started Session 12 of user kali.` |
| **/var/log/auth.log** | Authentication & authorisation: SSH logins, sudo, su, password changes. | – Contains **no** kernel, cron, mail, daemon events (unlike `syslog`).<br>– Focuses entirely on access security, useful for brute‑force detection. | `May 17 17:30:22 ubuntu1103 sshd[1234]: Failed password for invalid user test from 192.168.1.200 port 54321 ssh2`<br>`May 17 17:30:25 ubuntu1103 sudo:   kali : TTY=pts/0 ; PWD=/home/kali ; USER=root ; COMMAND=/bin/ls` |
| **/var/log/kern.log** | Kernel messages: drivers, hardware, memory errors, network activity at kernel level. | – Unlike `syslog` (which also contains kernel but less detail).<br>– Unlike `auth.log` (no authentication).<br>– Used for low‑level system/hardware diagnostics. | `May 17 14:32:45 ubuntu1103 kernel: [12345.678] usb 2-1: new high-speed USB device number 5 using ehci-pci`<br>`May 17 14:32:46 ubuntu1103 kernel: [12346.789] segfault at 7f8c9a1c ip 00007f8c9a1c sp 00007ffc2d8c error 4 in libc-2.35.so` |
| **/var/log/audit/audit.log** | Detailed audit log based on custom rules (auditd): file access, command execution, configuration changes, syscalls. | – **Not a default log** (requires `auditd` installation).<br>– Provides the highest granularity (down to syscalls).<br>– Unlike `auth.log` (only authentication results), `kern.log` (kernel), `syslog` (general). | `type=SYSCALL msg=audit(1747467894.123:456): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=7f8c9a1c a2=80000 a3=1b6 items=1 ppid=1234 pid=5678 uid=1000 gid=1000 euid=1000 suid=1000 fsuid=1000 egid=1000 sgid=1000 fsgid=1000 tty=pts0 ses=1 comm="cat" exe="/bin/cat" key="passwd_access"` |

---

## 💡 Core Differences Summary

| Log | Primary Scope | What Makes It Different |
|-----|---------------|--------------------------|
| **Application (Win)** | Software applications | No security or system events. |
| **Security (Win)** | Authentication, authorisation, security | Based on Audit Policy, not app/driver logs. |
| **System (Win)** | OS, drivers, services | Focuses on system stability, not apps or security. |
| **syslog (Linux)** | General system events | All‑purpose but less detailed than specialised logs. |
| **auth.log (Linux)** | Authentication, sudo, login | Only access security; no kernel/cron/daemon. |
| **kern.log (Linux)** | Kernel, drivers, hardware | More detailed kernel info than syslog; no authentication. |
| **audit.log (Linux)** | Syscalls, file access, command execution | Highest detail, requires separate installation; completely different from default logs. |

# 7. Wazuh
## Server-side: 
Wazuh - Installation : (Indexer - Cluster - Server - Dashboard )
<img width="1461" height="735" alt="image" src="https://github.com/user-attachments/assets/fb2f9b4f-e3d9-42cc-ac84-d4ce4d50c436" />
./wazuh-install.sh -i --wazuh-dashboard

Finally 
<img width="1501" height="1389" alt="image" src="https://github.com/user-attachments/assets/4a5a5308-b937-4134-8b85-c99f229c6657" />

Hitting Security -> Internal Users -> Create Internal User (tk: cybex; mk: wzhcybex)

## Client-side: 
Deploy a new agent - client - ubbuntu
<img width="984" height="1165" alt="image" src="https://github.com/user-attachments/assets/40066445-565a-4182-b6f3-f891d09c5fdd" />
Commanline in client - ubuntu 
<img width="1500" height="490" alt="image" src="https://github.com/user-attachments/assets/e9b1e80e-c42b-4f20-a763-4780db27122a" />

Successfully Download with 1 Active Ubuntu 
<img width="1519" height="520" alt="image" src="https://github.com/user-attachments/assets/3a09774f-08a2-480c-a8fd-08e68d6c19f1" />

Security Events
<img width="1527" height="1520" alt="image" src="https://github.com/user-attachments/assets/f0e668a9-94f4-4cff-b8de-24e12ff1757d" />

Deploy a new agent  - client - window
<img width="1580" height="1541" alt="image" src="https://github.com/user-attachments/assets/64a63691-f67a-4dec-80d5-5b5777b07502" />

Commandline in client - window 
<img width="976" height="249" alt="image" src="https://github.com/user-attachments/assets/668e7b67-2bd5-4b64-991c-a0cb5c386d28" />

Successfully Connection 
<img width="1522" height="668" alt="image" src="https://github.com/user-attachments/assets/7799eab0-8794-4fe7-8c51-dd7fc5701f7b" />


# 7. RAW LOG TO SIEM
<img width="1404" height="738" alt="image" src="https://github.com/user-attachments/assets/7d39f83d-30ce-4d38-a2e8-96a946828880" />

SYSLOG
0. Making log
<img width="649" height="47" alt="image" src="https://github.com/user-attachments/assets/98fd9862-fddf-4dce-a7af-58a260ec525f" />

1. RAW LOG : tail -n 2 /var/log/syslog
<img width="649" height="64" alt="image" src="https://github.com/user-attachments/assets/b9507309-3f47-4ae0-b7bd-cc278a825cad" />

2. Filebeat reads the new line
Filebeat (installed on the client) runs as a service, continuously monitoring the configured log files. When it detects a new line, it reads it and passes it to the processor.
3. Parsing – syntactic analysis
Filebeat applies a Grok pattern (a named regex) to extract components.

Default Grok pattern for syslog:

      text
      %{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:hostname} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}

Temporary result (non‑normalized fields):

    text
    timestamp: "May 19 15:30:45"
    hostname: "ubuntu11031"
    program: "ubuntu11031"
    message: "sshd: Failed password for invalid user admin from 192.168.1.100 port 54321 ssh2"

4. (Optional) Additional processor – extract details
If a dissect or custom grok for the "Failed password" line is configured, Filebeat further splits the message field:

-      text
       user.name: "admin"
       source.ip: "192.168.1.100"
       port: 54321
       failure: "Failed password"

5. Normalization according to ECS (Elastic Common Schema)

Filebeat and Elasticsearch automatically add or map fields to the Elastic Common Schema, ensuring consistent field names across different log sources.

- `@timestamp` ← convert `timestamp` from `May 19 15:30:45` to ISO8601 (with timezone).
- `host.name` ← `hostname`
- `process.name` ← `program` (if the program is `sshd`, it may be extracted separately)
- `user.name`, `source.ip` kept as is.
- `event.category`: assigned `authentication` (based on content)
- `event.type`: `denied`
- `event.outcome`: `failure`
- `log.file.path`: automatically added with the source file path.
- `agent.type`, `agent.version`, `ecs.version` added.

---

6. Package as JSON and send to Elasticsearch

Filebeat creates a complete JSON document and sends it via HTTP to Elasticsearch.

**Example document (abbreviated):**

```json
{
  "@timestamp": "2026-05-19T08:30:45.000Z",
  "host": { "name": "ubuntu11031" },
  "source": { "ip": "192.168.1.100" },
  "user": { "name": "admin" },
  "event": {
    "category": "authentication",
    "type": "denied",
    "outcome": "failure"
  },
  "message": "sshd: Failed password for invalid user admin from 192.168.1.100 port 54321 ssh2",
  "log": { "file": { "path": "/var/log/syslog" } },
  "agent": { "type": "filebeat", "version": "8.19.15" }
}

```
7. Elasticsearch stores and indexes
Elasticsearch receives the JSON, indexes each field, enabling extremely fast queries.
8. Kibana displays
When you go to Discover, Kibana sends a query to Elasticsearch and displays the results in a table. You can expand each row to see the separated fields.
<img width="1466" height="774" alt="image" src="https://github.com/user-attachments/assets/85657456-d1e7-44ba-88d1-a65a713032ab" />
