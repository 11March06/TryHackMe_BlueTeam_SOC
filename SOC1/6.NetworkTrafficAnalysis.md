# 1. Basics
## a. Introduction
Network Traffic Analysis (NTA) is the process of capturing, inspecting, and analyzing data as it moves across a network. It purposes is to gain full visibility into communications and understand whhat is normal versus suspicious behaviour.
NTA is not just like Wireshark - it involves comnbiming multiple data sources such as logs, packet inspection, and traffic statistics to detect anomalies and potential threats.
This skiils is essential for roles like SOC analysists, helping them identify unusual activity within large volumes of network data.

Learning objectives include : 
  - Understanding what NTA is
  - Knowing what network data can be observed
  - Learning how to monitor network traffic
  - Recognizing common traffic sources and flows

## b. Purpose of Network Traffic Analysis
  - This scenario shows attackers can abuse DNS traffic to hide malicious activity. A compromised system sends many DNS queries with different subdomains to the same domain, which is a sign of covert communication
  - Standard DNS logs only shows basic info (queries, IPs, timestamps) but not the content. By performing network traffic analysis, we can inspect DNS packets and uncover hidden data, such as C2 instructions embedded in TXT records.
  - This technique is often used for:
  -   - Data exfiltration
      - Remote commnand execution
      - Stealthy communication with attacker servers
For example : <img width="1241" height="707" alt="image" src="https://github.com/user-attachments/assets/b460fa8e-20f5-4e80-8483-dbee44c96420" />

--> NTA is used to gain visibility into network activity and identify potential threats. It helps :
  - Monitor performance: detect slowdowns or unusual traffic spikes
  - Identify abnormalities: spot deviations from normal behaviour
  - Inspect suspicious communication: detect data exfiltration, malware downloads, or lateral movement
From a SOC perspective. NTA is essential for:
  - Detecting malicious activity
  - Reconstructing attacks during incident response
  - Validating security alerts
Real-world examples include detecting malicious HTTP downloads and identifying DNS tunneling used for data exfiltration.

## c. What Network Traffic Can We Observe?
<img width="1423" height="521" alt="image" src="https://github.com/user-attachments/assets/fb45a3d3-a483-48b0-8234-691c61e51630" />
 * TCP/IP Layers & What We Can Observe
    1. Application
       - Contains protocols like HTTP, DNS
       - Logs show headers (GET request, Host, etc.)
       - *Payload**  (e.g : actual ZIP file content) is usually NOT visible in logs
    2. Transport Layer (TCP/UDP)
       - Break data into segments and adds headers
       - Logs include:
           - Source/Destination ports
           - Flags (SYN/ ACK)
       - Important field:
           - Sequence Number --> Used to detect session hijacking
    3. Internet Layer (IP)
        - Adds IP addresses (Sources/ Destination) and TTL
        - *Useful for detecting**
            - Fragmentation attacks (used to evade IDS by splitting packets)
    4. Link Layer
       - Uses MAC addresses
       - *Helps detect** :
           - ARP spoofing/ poisoning attacks
        
* **Key Takeaways**
  - Logs only show partial data --> full packet analysis is needed (NTA is needed)
  - Common attack techniquess:
    - DNS tunneling (C2 communication)
    - Fragmentation (IDS evasion)
    - Session hijacking (TCP abuse)
    - ARP spoofing (network manipulation)

## d. Network Traffic Sources and Flows
  1. Traffic Sources
     There are 2 main categories:
     - Intermediary Devices
       - Devices that traffic passes through
       - Examples : firewall, router, switch, IDS/IPS, proxy
       - Generate less traffic:
         - Routing (OSPF, BGP)'
         - Management (SNMP, PING)
         - Support (ARP, DHCP)
     - Endpoint Devices:
       - Devices where traffic starts and ends
       - Examples: PC, server, phone, IoT, VM
       - Generate most of the network traffic
  2. Network Traffic Flows
    * North-South (NS) Traffic:
      - Traffic in/out of the network (LAN <--> Internet)
      - Passes through firewall
      - Common protocols:
        - HTTPS, DNS, SSH, VPN, SMTP, RDP
      - Includes:
        - Inbound (ingress)
        - Outbound (engress)
    --> Highly monitored
    * East-West (EW) Traffic:
      - Traffic inside the network (LAN <-> LAN)
      - Often less monitored
      - Important for detecting:
        - Lateral movement (attacker spreading inside network)
      - Includes services:
        - Active Directory, SMB, internal apps, backups

  3. Example Flows
* HTTPS (with proxy)
  Client ↔ Proxy ↔ Web Server
  Proxy inspects traffic (TLS inspection)
→ Actually 2 sessions, not 1
* DNS Flow
  Client → Internal DNS → External DNS → back
  Internal DNS acts on behalf of client
* SMB with Kerberos
  User authenticates via Kerberos (Domain Controller)
  Gets ticket → then connects to file share (SMB)

  🔹 Key Takeaways
  - Endpoints = main traffic source : category of devices which generates the most traffic in a network
  - North-South = external traffic (important for security)
  - East-West = internal traffic (important for detecting spread)
  - Attacker get in by NS , spread by EW
  - Many attacks happen inside network after initial breach
  - TLS - Transport Layer Security


## e. How Can We Observe Network Traffic
<img width="812" height="605" alt="image" src="https://github.com/user-attachments/assets/3e54c931-d882-4f0c-97b7-b4950d83990f" />

1. Logs
  - The most basic source
  - Available on almost all systems (Linux, Windows, Web servers)
  - Examples :
    - Source/ Destination IP
    - HTTP requests
    - Login events (SSH)
  - **Limitation** : Logs do NOT include full packet data (no payload)

2. Full Packet Capture (PCAP)
  --> Provides complete visibility of packets
  - Methods:
    - Network TAP
      - Physical Device
      - Copies traffic directly
      - No performance impact
    - Port Mirroring (SPAN)
      - Cofigured on switches
      - May impact performance under heavy load
    --> Used for: Malware analysis + Inspecting real content (HTTP, DNS, files)

  3. Network Statistics
-- > Focus on metadata, not full packets
     Examples:
      - Number of DNS requests
      - Communication patterns between hosts
     Technologies:
      - NetFlow
      - IPFIX
    --> Used to detect:
        * Command & Control (C2)
        * Data exfiltration
        * Lateral movement
  * **Key Idea**
    - Logs --> Overview
    - PCAP --> Full details
    - NetFlow/IPFIX --> behaviour & patterns
      --> Best results come from combining all three
    
