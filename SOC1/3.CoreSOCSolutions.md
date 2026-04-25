# 1. Introduction to EDR
## a. Intorduction
  - Understand the basics of EDR and how it works
  - Differentiate EDR from traditional Antivirus solutions
  - Examine the architecture of an EDR solution
  - Analyze the types of telemetry it collects from endpoints
  - Understand the detection and response capabilities of an EDR
  - Investigate a realistic alert in the EDR
## b. EDR 
* Endpoint Detection and Response (EDR) is a security solution that offers deep-level protection for endpoints. No matter where the endpoints are, the EDR will make sure they are monitored constantly and threats are detected.
  ** Features of EDR**
-->  There are three main features of an EDR, wwhich can also be referred to as the three pillars of an EDR solution
<img width="835" height="503" alt="image" src="https://github.com/user-attachments/assets/a53ebd51-cfee-4a7c-b58e-7e1800fa3468" />

* Visibility
  - A good analysis often depends on the available level of visibility of the activity
  - This is one of the features of EDR that makes it unique from other endpoint security solutions
<img width="1278" height="168" alt="image" src="https://github.com/user-attachments/assets/b9bacd6f-3088-4c98-b566-843ec1440bbe" />

  - Each node represents a process
<img width="1404" height="445" alt="image" src="https://github.com/user-attachments/assets/11ea6b31-ebcc-464d-bd55-c8771487ccf1" />

--> EDR provides deep visibility into endpoint activity, including processes, registry changes, file modifications, and user actions. It organizes this data into a structured format, allowing analysts to view full process trees and timelines. This helps analysts understand attack behavior, investigate incidents, and perform threat hunting with complete context.
    
* Detection
<img width="1391" height="537" alt="image" src="https://github.com/user-attachments/assets/227589db-1f35-40f0-bee8-b3729b777a77" />

--> EDR detection is more advanced than traditional methods by combining signature-based and behavior-based techniques. It uses machine learning to detect anomalies, including fileless malware in memory, and supports custom IOCs. Detections are displayed in a dashboard with details like severity, time, file, host, and user, and are mapped to MITRE techniques, providing rich context for analysis.

* Response
<img width="1393" height="629" alt="image" src="https://github.com/user-attachments/assets/430d0221-d5ce-4dd0-bab4-5aa2d8b90723" />

--> EDR allows analysts to respond to threats directly from a central console. They can take actions such as isolating endpoints, terminating processes, quarantining files, or remotely accessing a host to investigate and respond. With strong visibility, detection, and response capabilities, EDR is a powerful tool, but it is limited to endpoint-level monitoring and does not detect network-based threats.

**Summary:**
EDR = Visibility + Smart detection + Fast response on endpoints
But it cannot replace network security solutions

**Limitation:**
EDR only provides protection at the host level.
It does not detect network-level attacks.

**Note** : Visibility is the feature of EDR provides a complete context for all the detections

## c. Beyond the Antivirus
**Why do we need an EDR when we already have an Antivirus (AV) on the endpoints?* 

* Both of these security solutions have the same motive of protecting the endpoint on wwhich they are installed. 
* Antivirus (AV) mainly uses signature-based detection to block known threats. It works well for already identified malware but can miss new or advanced attacks.

* EDR goes further by:
  - Monitoring behavior and activity on the endpoint
  - Providing full visibility and context (processes, timelines, actions)
  - Detecting unknown or advanced threats (e.g., fileless attacks, obfuscated scripts)
  - Allowing real-time response (isolate host, kill process, etc.)
  - Giving organization-wide visibility across all endpoints

* **Simple analogy** 
  - AV = Immigration check (blocks known criminals)
  - EDR = Security inside the airport (monitors behavior and reacts to suspicious activity)

 <img width="1455" height="666" alt="image" src="https://github.com/user-attachments/assets/6dfa55dc-15c8-418e-aaa7-07afd7aeaf14" />
<img width="1764" height="1240" alt="image" src="https://github.com/user-attachments/assets/5046a64d-c603-4e10-a43a-a42066abf8d6" />
* Note: Some modern AVs may have more enhanced visibility and detection. However, an EDR is ahead as it levels up the detection and response in an endpoint.

## d. How an EDR work?

<img width="1398" height="674" alt="image" src="https://github.com/user-attachments/assets/c712fc1e-3e86-4cdc-a810-2f9b1eabd89e" />
**Agents** (Sensor):  Installed on endpoints to monitor activities (processes, files, registry, network) and send data in real time.
**EDR Console**: Collects and analyzes data using logic, machine learning, and threat intelligence to generate alerts
**After Detection**: Analysts prioritize alerts by severity (critical -> low), investigate details, and decide if they are True Positive or False Positive. If malicious, they take response actions.
**Integration**: EDR works with other security tools (Firewall, DLP, IAM, etc.) and is often integrated into a SIEM for centralized monitoring.

## e. EDR Telemetry
* Telemetry is the data collected by EDR from endpoints to monitor and detect activity. It acts like a “black box”, recording everything happening on a system.

* Collected Telemetry
  - Processes: Tracks running/stopped programs → detects suspicious behavior
  - Network: Monitors connections → finds malicious or unusual traffic
  - Command line: Logs CMD/PowerShell commands → catches harmful scripts
  - Files/Folders: Tracks changes → detects malware or ransomware activity
  - Registry: Monitors system changes → identifies persistence techniques

 *Purpose:
  - Detect threats
  - Support investigation
  - Reconstruct attack timelines
 
--> In short: Telemetry = detailed data that helps identify and investigate suspicious activity on endpoints.

## f. Dectection and Response Capabilities
<img width="738" height="1075" alt="image" src="https://github.com/user-attachments/assets/a140e8af-c7c8-4a5e-afa5-4bbbd1ac9027" />

* **Detection**
EDR uses telemetry data to detect threats with advanced techniques:
* Behavioral Detection: Identifies suspicious behavior instead of relying only on known signatures
→ Example: Microsoft Word launching PowerShell
* Anomaly Detection: Flags activities that deviate from normal system behavior
→ May produce false positives but gives useful context
* IOC Matching: Compares activity with known threat indicators (hashes, IPs, etc.), based on known malicious behaviours.
→ Quickly detects known attacks
* MITRE ATT&CK Mapping: Maps detected activity to attack tactics and techniques
→ Helps analysts understand attack stages
* Machine Learning: Detects complex or hidden attack patterns
→ Useful for fileless and multi-stage attacks

* **Response**
After detection, EDR enables actions to stop or investigate threats:
* Isolate Host: Disconnect infected machine from network to stop spread
* Terminate Process: Stop malicious processes without isolating the system
* Quarantine: Move malicious files to a safe location
* Remote Access: Analysts can access and control the endpoint remotely
* Artefact Collection: Gather data for investigation (memory dump, logs, registry, etc.)

**KEY IDEA** : EDR goes beyond traditional antivirus by combining advanced detection + powerful response tools, helping analysts detect, contain, and investigate modern cyber attacks effectively.

# 2. Introduction to SIEM
## a. Introduction
* **Security Information and Event Management system** (SIEM) is the core security solution that a SOC analyst uses in the security operations center
- Understand the different types of log sources
- Identify the limitations of working with isolated logs
- Recognize the importance of a SIEM solution
- Explore the features of a SIEM solution
- Learn various types of log sources and their ingestion in the SIEM
- Understand the process behind alerting and alert analysis

## b. Logs Everywhere, Answers Nowhere
* **Logs Everywhere** 
<img width="926" height="792" alt="image" src="https://github.com/user-attachments/assets/9c46471a-8087-4448-b796-7b14a37068ef" />
- Overview: In a network, devices (computers, servers, routers) continuously generate logs that record their activities -> These devices are called log sources
  -> Logs are important for:
  - Detecting threats
  - Investigating incidents
  - Troubleshooting systems
  
i. Host-Centric Logs Sources: Logs that capture activities inside a device (endpoint/server). Devices that generate host-centric logs include Windows, Linux, servers, etc. Some examples of host-centric logs are:
  - File access by a user
  - Login/authentication attempts
  - Process execution
  - Registry changes
  - PowerShell activity

  --> Sources: Windows, Linux, servers
  
ii. Network-Centric Log Sources: Logs that capture communication between devices or with the Internet. Devices that generate network-centric logs are firewalls, IDS/IPS, routers, etc. Some examples of network-centric logs are:
  - SSH connections
  - FTP file access
  - Web traffic (HTTP/HTTPS)
  - VPN access
  - Network file sharing
      --> Sources: Firewalls, IDS/IPS, routers
 
  **Key Idea**
--> Both host-centric and network-centric logs together provide a complete view of activity in a network, helping detect and investigate security issues effectively.
   
**Answers Nowhere** 
Challenges with Log Analysis .Even though logs are useful, working with them has several difficulties:

* Too Many Log Sources
  - Networks have many devices generating hundreds of logs per second.
  - Logs are spread across multiple systems, making investigation slow and overwhelming.

* No Centralization
  - Logs are stored on individual machines.
  - Analysts must connect to each device (via SSH, RDP, etc.) to access logs, which is inefficient and time-consuming.

* Limited Context

  - A single log does not provide the full picture.
  - Only by correlating logs from different sources can analysts understand the real situation.

Example: A file access may seem normal, but when combined with other logs, it could indicate lateral movement after a compromise.

* Limited Analysis Capability
  - The large volume of logs makes manual analysis nearly impossible.
  - Analysts can easily miss important events.

* Format Issues
  - Different systems generate logs in different formats.
  - This makes it difficult for analysts to understand and analyze them consistently.

**Key Idea**:
--> Logs are important, but without proper tools and centralization, they are difficult to manage and analyze effectively.

## c. Why SIEM?
<img width="557" height="504" alt="image" src="https://github.com/user-attachments/assets/f7923745-5411-49b5-97ca-46fb27648bd2" />
* SIEM (Security Information and Event Management) helps manage large volumes of logs and turn them into useful security insights by:

  - Centralizing logs from multiple sources into one place
  - Normalizing data into a consistent format for easy analysis
  - Correlating events to detect suspicious patterns across systems
  - Using detection rules to identify potential threats automatically
  - Providing real-time alerts for quick response
  - Displaying dashboards for clear visualization and reporting

-->  In short, SIEM reduces noise, improves visibility, and helps security teams detect and respond to threats more efficiently.

## d. Log Sources and Ingestion
 * Log Sources
- Windows: Uses Event Viewer, each event has a unique Event ID
- Linux: Logs stored in /var/log/:
    - auth.log / secure: authentication
    - cron: scheduled jobs
    - httpd: web server logs
    - kern: kernel events
- Web Server (Apache):
    - Records requests/responses (IP, URL, status code, user-agent)
    -Useful for detecting web attacks

* Log Ingestion

Ways to send logs to SIEM:
- Agent/Forwarder: Installed on endpoints, sends logs automatically (most common, e.g. Splunk)
- Syslog: Standard protocol for real-time log transfer
- Manual Upload: Upload logs for quick/offline analysis
- Port Forwarding: Devices send logs to a listening SIEM port

* Importance
- Logs help detect security issues and attacks
- SIEM functions:
  - Collect
  - Normalize
  - Analyze
  - Alert
  -->  Correlating logs reveals full attack patterns
## e. Alerting Process and Analysis
* How SIEM triggers alerts
- SIEM uses detection rules
- Rules are logical IF–THEN conditions based on logs
Examples:
- Multiple failed logins → brute-force alert
- Successful login after failures → suspicious
- USB insertion → alert if restricted
- High outbound traffic → possible data exfiltration
- How detection rules are created

Use-case 1: Log deletion
- Event ID 104 indicates log clearing
Rule:
If LogSource = WinEventLog and EventID = 104 → trigger alert

Use-case 2: whoami command

- Event ID 4688 indicates process creation
Rule:
If LogSource = WinEventLog and EventID = 4688 and ProcessName contains "whoami" → trigger alert

* Note:
- Rules depend on field-value pairs
- Normalized logs are important
- Alert investigation

* When an alert is triggered:
- Check SIEM dashboards (e.g. Splunk)
- Analyze related logs
- Review the triggered rule

Outcomes:
- False Positive → tune the rule
- True Positive → take action:
  - Investigate further
  - Contact asset owner
  - Isolate infected host
  - Block malicious IP
  
