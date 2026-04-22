# 1. SOC L1 Alert Triage
## a. Introduction
- An alerts is a core concept for any SOC team
- Understanding the concepts and lifecycle of alerts, from event generation to collect resolution
## b. Events and Alerts 
* Imagining yourself as a SOC intern, looking at the monitor of your mentor, a SOC L1 or L2 analyst, and seeing hundreds of alerts appear.
 
* Alert Management Platforms
<img width="1224" height="547" alt="image" src="https://github.com/user-attachments/assets/e4a233b5-df29-4179-b1bc-19c6f187e34d" />

* L1 Role in Alert Triage
- SOC L1 analyst: Review the alerts, distinguish bad from good, and notify L2 analyst in case of a real threat
- SOC L2 analyst: Receive the alerts escalated by L1 analysts and perform deeper analysis and remediation
- SOC engineer: Ensure the alerts contain enough information required for efficient alert triage
- SOC manager: Track speed and quality of alert triage to ensure that real attacks won't be missed

## c. Alert Properties
- While the details differ for every SIEM or security solution, the alerts have generally have a few main properties you must understand before taking them.
<img width="1494" height="484" alt="image" src="https://github.com/user-attachments/assets/e2f084dd-fcc3-4b83-8331-353adfab7bdf" />

- Alert Properties
<img width="1108" height="1154" alt="image" src="https://github.com/user-attachments/assets/8eb56be5-236c-42fd-b910-9055bbaaef76" />

## d. Picking Right Alert
- Every SOC team decides on its own prioritisation rules and usually automates them by setting the appropriate alert sorting logic in SIEM or EDR.
  1. Filter the alerts
     --> Make sure you don't take the alert that other analysts have already reviewed, or that is already being investigated by one of your teammates. You should only take news, yet unseen and unresolved alerts.
  2. Sort by severity
     --> **Start with critical alerts, then high, medium, and finally low.** This is because detection engineers design rules so that critical alerts are much more likely to be real, major threats and cause much more impact than medium or low ones.
  3. Sort by time
     --> Start with the oldest alerts and end with the newest ones. The idea is that if both alerts are about two breaches, the hacker from the older breach is likely already dumping your data, while the "newcomer" has just started the discovery.

## e. Alert Triage
- Alert review by SOC analysts can also be called alert triage, alert processing, alert investigation, or alert analysis.
* Sticking to the **Alert Triage** 
<img width="1231" height="428" alt="image" src="https://github.com/user-attachments/assets/1c168900-390d-4461-8e46-3c9ec26c270d" />

* **Initial Actions**
  - Initial strps are designed to ensure taking ownership of the assigned alert and avoid interfacing with alerts being handled by other analysts, and confirm being fully prepared to proceed with the detailed investigation.
  - Achieving it by first assigning the alert to yourself, moving it to **IN PROGRESS**, and then familliarising yourself with the alert details like its name, description, and key indicators.

* **Investigation**
  - The most complex step, requiring to apply technical knowledge and experience to understand the activity and analyse its legitimacy in SIEM or EDR logs.
    1. Understand who is under threat, like the affected user, hostname, cloud, network, or website
    2. Note the action described in the alert, like whether it was a suspicious login, malware, or phishing
    3. Review surrounding events, looking for suspicious actions shortly after or before the alert
    4. Use threat intelligence platforms or other available resources to verify your thoughts
   
* **Final Actions**
  -  Decide whether the alert is a True Positive (malicious) or False Positive (benign).
  -  Write a detailed comment explaining your analysis and reasoning. Then return to the dashboard and set the alert status to "closed"
 
* **NOTE**:
  - Virustotal, Whois Lookup --> Look up, scan, investigate file, host, url, domain, etc..

# 2. SOC L1 Alert Reporting
## a. Introduction 
- Understand the need for SOC alert reporting and escalation
- Learn how to write alert comments or case reports properly
- Explore escalation methods and communication best practices
- Apply the knowledge to triage alerts in a simulated environment
- Feel more confident in SOC Simulator and during SAL1 certification
## b. Alert Funnel 
- L1 analysts receive the alerts in a SIEM, EDR, or a ticket management platform.
- Most of the alerts are closed as False Positives or are handled on L1 level, but complex and threatening ones are sent to L2 that remediate most breaches.
- New terms : reporting, escalation, and communication.
<img width="971" height="449" alt="image" src="https://github.com/user-attachments/assets/26cbfb04-4e97-4ece-b94e-8184ccc4e3d8" />

* **Alert Reporting**
  - The process of formally describing alert details and findings
  - Depending on team standards and alert severity, instead of a short alert comment, you can be required to document your investigation in detail, ensuring all relevant evidence is included.
    --> This is especially important for True Positives, which require escalation

* **Alert Escalation**
  - The process of passing suspicious alerts to an L2 analyst for review.
  - If the True Positive alert requires additional actions or deeper investigation, escalate it to the L2 analyst for further review following the agreed procedures.

* **Communication**
  - May also need to communicate other departments during or after the analysis.

## c. Reporting Guide
- It is essential for L1 analysts to write reports in addition to making them as True or False Positives
- Having L1 analysts write alert reports serves several key purpose: 
<img width="1326" height="1022" alt="image" src="https://github.com/user-attachments/assets/e2902db1-4942-49fe-8c64-012ce965e62b" />

**Report Format**
Recommand following the **FIVE Ws** approach and including at least these items in the report:
* **Who**: Which user logs in, runs the command, or downloads the file
* **What**: What exact action or event sequence was performed
* **When**: When exactly did the suspicious activity start and ended
* **Where**: Which device, IP, or website was involved in the alert
* **Why**: The most important W, the reasoning for your final verdict

Example : <img width="1851" height="563" alt="image" src="https://github.com/user-attachments/assets/5101a8ac-9e40-459e-a612-34d3fb2c0e2b" />

## d. Escalation Guide

**Recommandations** 
 1. The alert is an indicator of a major cyberattack requiring deeper investigation or DFIR
 2. Remediation actions like malware removal, host isolation, or password reset are required
 3. Communication with customers, partners, management, or law enforcement agencies is required
 4. You just do not fully understand the alert and need some help from more senior analysts


**Escalation Steps** 
- To escalate the alert, in most cases, all you have to do is to reassign the alert to the L2 on shift and ping them in corporate chat or in person. In some teams though, you may be required to create a formal written escalation request with dozens of required fields.
- No matter what the agreements are, L2 will eventually receive the ticket from you, read your report, and contact you in case of any questions. Once everything is clear, the L2 analyst will typically research the alert details further, validate if the alert is indeed a True Positive, communicate with other departments if needed, and, for major incidents, start a formal Incident Response process.
<img width="3010" height="872" alt="image" src="https://github.com/user-attachments/assets/798cea2c-7cc1-4693-a558-775265811e04" />

**Requesting L2 Support** 
- It is generally fine for L1 to request senior support if something is unclear. Especially in your first months, it's always better to discuss the alert and clarify SOC procedures than to blindly close the alert you don't understand yourself. The procedures for requesting support may differ, but the flow generally looks like this:
<img width="3014" height="772" alt="image" src="https://github.com/user-attachments/assets/6ce99a6a-5577-4b91-8478-e504ee77ea2b" />

## e. SOC Communication 
- The escalation and reporting topics should sound straightforward and logical
- Should be prepared for unexpected scenarios and know what to do in critical cases. 


