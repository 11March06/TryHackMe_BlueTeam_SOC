#1 . Intro to Cyber Threat Intel
## a. Introduction 
A SOC is a high-volume environment where Level 1 (L1) analysts handle the first layer of alert triage and investigation.
Cyber Threat Intelligence (CTI) helps analysts move beyond raw indicators (like IPs or hashes) to understanding the context, intent, and risk behind threats.
The goal for L1 analysts is to evolve from simply recognizing indicators to understanding why they are suspicious and being able to justify that reasoning.
Key CTI concepts include:
What threat intelligence is and why it is important in SOC operations
The threat intelligence lifecycle (collecting, processing, analyzing, and sharing intelligence)
Understanding Indicators of Compromise (IOCs) and other threat indicators
Using threat intelligence feeds and platforms to enrich alerts and improve detection
Overall, CTI helps SOC analysts:
Reduce noise from alerts
Improve decision-making during triage
Respond faster and more accurately to real threats

In short, CTI transforms SOC work from reactive alert handling into context-aware threat analysis and decision-making.

## b. Cyber Threat Intelligence
<img width="748" height="702" alt="image" src="https://github.com/user-attachments/assets/8f949cca-a714-43af-9f3e-56ee68c8422f" />
In a SOC, Level 1 analysts face large volumes of alerts and must quickly decide what is real and what is noise. CTI provides the context needed to prioritise correctly.


CTI helps answer three key questions:


Who/what is behind the indicator?


What have they done before?


What should we do right now?




The main goal is to turn raw alert data into actionable intelligence, reducing guesswork and improving response speed.



Data → Information → Intelligence


Data: Raw values (e.g. IP address)


Information: Data with context (e.g. IP reputation, ownership)


Intelligence: Analysed insight that drives action (e.g. confirmed C2 server → block immediately)


L1 analysts play a key role in enriching alerts so they can move up this ladder.

Key CTI Concepts


IOC (Indicator of Compromise): Evidence of compromise (e.g. malicious IP, hash)


IOA (Indicator of Attack): Suspicious behaviour in progress (e.g. PowerShell misuse)


TTPs: Adversary methods and patterns mapped to frameworks like MITRE ATT&CK



Indicator Enrichment Examples


IP / Domain / URL / File hash / Email / Registry key


Enrichment sources include:


WHOIS, VirusTotal, Shodan


URL reputation tools (URLhaus, urlscan.io)


Sandboxes and EDR tools





CTI Infrastructure


Feeds: Streams of indicators (can include false positives if not curated)


Platforms: Systems like MISP or OpenCTI that store and correlate intelligence


Good CTI practice = careful feed selection + centralised platform for trusted data


Intelligence Sources


Internal telemetry: SIEM, EDR, logs (most valuable)


Commercial feeds: High quality but licensed


OSINT: Public sources (must be verified)


Communities / ISACs: Sector-specific threat sharing groups



CTI Types


Strategic: Long-term risk trends for business decisions


Tactical: Attacker TTPs and techniques


Operational: Campaign-level intent and targeting


Technical: Raw indicators (IPs, hashes, domains)



Key takeaway
CTI transforms SOC L1 work from reactive alert handling into context-driven decision-making, enabling faster, more accurate triage and better escalation decisions.
<img width="816" height="807" alt="image" src="https://github.com/user-attachments/assets/201dd6ce-3eb9-4735-98f0-1de98f2c09f1" />

## c. CTI Lifecycle
The Cyber Threat Intelligence (CTI) lifecycle is a 6-step process that turns raw threat data into actionable security decisions for SOC analysts.

1. Direction: Define goals and intelligence requirements. Analysts decide what they need to find out (e.g. which IPs or malware affect PostgreSQL).

2. Collection: Gather raw intelligence from sources like:
OSINT (AbuseIPDB, public reports)
Commercial feeds
Internal systems (SIEM, EDR, MISP)

3. Processing: Convert raw data into usable formats by:
Normalising and standardising indicators
Removing duplicates
Correlating across sources
Preparing outputs (e.g. blocklists, YARA rules)


4. Analysis: Evaluate relevance and credibility of indicators, reduce false positives, and decide what is truly actionable.

5. Dissemination: Share intelligence with the right teams (firewall, EDR, management) in the correct format and TLP level.

6. Feedback: Measure effectiveness (e.g. reduced dwell time, fewer incidents) and refine future intelligence cycles.

* Key idea:
CTI is a continuous loop, helping SOC L1 analysts turn scattered threat data into structured, actionable defence decisions.

## d. CTI Standards & Frameworks
<img width="1748" height="953" alt="image" src="https://github.com/user-attachments/assets/5ecc4bce-fd8f-41ac-92de-1624ef41f254" />
MITRE ATT&CK
A global knowledge base of adversary behaviours (TTPs).
Organises attacks into tactics and techniques (e.g. T1059.001 PowerShell).
Helps SOC L1 analysts:
Label suspicious activity in alerts
Standardise reporting
Enable faster understanding for L2/IR teams
MITRE D3FEND
The defensive counterpart to ATT&CK.
Focuses on how to defend against attack techniques.
Maps attacker behaviours to mitigation strategies (e.g. DNS tunneling → DNS request analysis).
Helps analysts suggest practical security controls, not just detection.
Cyber Kill Chain (Lockheed Martin)
A model that breaks an attack into sequential stages (from reconnaissance to impact).
Helps analysts understand:
Where an attack is in progress
What actions to expect next
How to interrupt the attack chain early
<img width="1717" height="983" alt="image" src="https://github.com/user-attachments/assets/1cbe254d-3594-43b7-ba67-4c3817e915fd" />

CVEs, CVSS, NVD, and Threat Intelligence Sharing
CVE (Common Vulnerabilities and Exposures)
A unique ID for a known software vulnerability (e.g., CVE-2023-4863).
Used as a standard reference so everyone talks about the same flaw.
CVSS (Common Vulnerability Scoring System)
A 0–10 severity rating system for vulnerabilities.
Helps SOC analysts prioritise issues based on impact and exploitability.
NVD (National Vulnerability Database)
A central repository that expands CVEs with:
CVSS scores
Affected products
Exploit details and metadata
Threat Intelligence Sharing Standards
STIX (Structured Threat Information Expression)
A structured JSON format for representing threat intelligence in a machine-readable way.

TAXII (Trusted Automated eXchange of Indicator Information)

A secure protocol/API used to share threat intelligence automatically between systems.

It supports:

Collection model: Pull intel from a provider
Channel model: Receive pushed intel from a central source
Key idea
CVE/CVSS/NVD help SOC teams understand and prioritise vulnerabilities
STIX/TAXII enable standardised, automated sharing of threat intelligence
Important caution

Not all intelligence should be shared freely—legal restrictions, NDAs, and operational security concerns may limit what can be disclosed, and early sharing can sometimes expose detection to attackers.

## e. Pracical Analysis

# 2. FIle and Hash Threat Intel
## a. Introduction

## b. Filenames and Paths
Filepaths and filenames can reveal attacker behaviour even before deeper analysis.
Common suspicious locations include:
  C:\Windows\Temp\ (temporary execution)
  C:\ProgramData\ (persistence and stealth storage)
  C:\Users\Public\ (cross-user accessibility)


Attackers also use filename tricks to avoid detection:
  Double extensions (e.g. invoice.pdf.exe)
  System binary impersonation (e.g. scvhost.exe mimicking svchost.exe)
  High-entropy/random names (e.g. jh8F21.exe)
  Masquerading names (e.g. backup-2300.exe)

--> In this case, Setup.exe matches the system binary impersonation indicator because it mimics a legitimate installer name often trusted by users.

## c. File Hash Lookup
File hashes (especially SHA256 and MD5) uniquely identify a file regardless of its name or location. This makes them a reliable way to track malware even when attackers rename or move it.
SOC analysts use tools like:
  certutil (Windows) or sha256sum (Linux) to generate hashes.
  VirusTotal to analyze hashes and get intelligence such as:
    Detection scores across antivirus vendors
    Threat labels and malware families
    File metadata (size, type, timestamps)
    Network indicators (domains/IPs)
    Behavioral analysis and dropped files
Key analysis checks in VirusTotal:
  High detection ratio = strong malware confidence
  Strange timestamps, high entropy, or missing signatures = suspicious
  Known malicious infrastructure or behaviors = confirmation of threat
MalwareBazaar complements this by:
  Providing malware samples and family tags
  Linking campaigns and threat actors
  Supplying YARA rules for detection
  Enabling deeper sandbox re-analysis
Key idea: --> File hashes act as an immutable fingerprint of malware, allowing SOC analysts to reliably identify threats even when everything else (name, path, disguise) changes.

## d. Sandbox Analysis

## e. Threat Intelligence Challenge


