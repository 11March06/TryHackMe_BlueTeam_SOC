# 1. Web Security Essentials 
## a. Introduction 
  - The internet is behind many aspects of modern life, from banking and shopping to social media and beyond As a result, websites and web applications are among attackers' most targeted assets. 
  - Learning :
    - Understand the shift from desktop applications to web applications
    - Learn why web applications are common targets for attackers
    - Explore web infrasstructure and the tools we use to protect the web
    - Practice applying security measures to harden a new web application

## b. Why WEB ?
  - Applications have shifted from desktop → web-based over time
  - Reasons:
    - Better accessibility (anywhere via browser)
    - Faster updates
    - Cloud computing & SaaS growth
<img width="1270" height="347" alt="image" src="https://github.com/user-attachments/assets/8fae7996-9d27-46fa-b363-4c52b6d6a32e" />
  - Security Perspective
  - Web apps are:
    - Always online
    - Publicly accessible
    - Connected to backend systems (databases, infrastructure)
→ This makes them a major attack target

* Risks
  - For owners:
    -Must secure systems 24/7
    -Responsible for protecting user data
    -Hard to keep up with new threats

  - For users: 
    - Risk of data theft, identity theft
    - Browser compromise = all accounts at risk
    -Privacy loss

* Real-world examples
    - Equifax (2017) → exploited web vulnerability → massive data breach
    - Capital One (2019) → WAF misconfiguration → exposed sensitive data
<img width="1342" height="986" alt="image" src="https://github.com/user-attachments/assets/d3e37b73-577f-4728-a0f1-ded08fafa28a" />

* Key Idea
  - Web apps bring convenience but also increase attack surface
  - Security responsibility mainly lies with the web app owner

## c. Web Infrastructure
  - Web works using a request–response cycle: 
    - Browser sends a request
    - Server processes it and sends back a response

  - Components of a Web Service
    - Application → website code (HTML, CSS, backend logic)
    - Web Server → handles requests and responses
    - Host Machine → OS/environment running everything
 <img width="1259" height="259" alt="image" src="https://github.com/user-attachments/assets/2a24877f-9d80-49c1-91d3-5f22aa81f87c" />
 
  - Web Servers
    - Common types:
      - Apache → popular for WordPress
      - Nginx → high performance
      - Internet Information Services → enterprise use
<img width="1277" height="209" alt="image" src="https://github.com/user-attachments/assets/68c115e5-65f9-4004-984b-e6087360f75b" />

* Key Idea
  - Web servers are public-facing → common attack targets
  - Understanding request flow helps detect attacks

## d. Protecting the Web
  - Some solutions visibility, while others can actively stop or limit an attack, commonly known as **mitigation**
  - Web security focuses on protecting 3 components:
    * Application
    * Web Server
    * Host Machine

Protection Measures
  - Application
    - Secure coding practices
    - Input validation & sanitization (prevent injections)
    - Access control (role-based permissions)

  - Web Server
    - Logging (record all requests)
    - Web Application Firewall (WAF) to block malicious traffic
    - CDN to reduce direct exposure

  - Host Machine
    - Least privilege (avoid admin-level processes)
    - System hardening (disable unused services/ports)
    - Antivirus / endpoint protection

General Security Practices
  - Strong authentication
  - Regular patching (app, server, OS)

Logging (Important for SOC)
  - Web servers generate access logs containing:
    - IP address
    - Timestamp
    - Requested resource
    - Response status
    - User agent

Example Normal Behavior
      1. User visits /index.html (GET)
  2. Goes to /login.html (GET)
  3. Submits credentials (POST)
  4. Accesses /myaccount.html

Key Idea
  - Logs help analysts:
    - Track user activity
    - Detect suspicious behavior
    - Reconstruct attack timelines
<img width="1222" height="255" alt="image" src="https://github.com/user-attachments/assets/70ecf9fe-fda2-4456-b74e-5ce052845f62" />

## e. Defense Systems
 * Content Delivery Network (CDN)
  - Distributes content via global edge servers
  - Improves speed and adds a security layer
  - Key benefits:
    - Hides origin IP (IP masking)
    - Protects against DDoS
    - Enforces HTTPS
    - Often includes built-in WAF
<img width="1281" height="481" alt="image" src="https://github.com/user-attachments/assets/45b172f9-11fa-4491-92be-926de1c6fc77" />

 * Web Application Firewall (WAF)
  - Filters and inspects HTTP requests
  - Blocks malicious traffic based on rules
  - Types:
    - Cloud-based (reverse proxy)
    - Host-based
    - Network-based
  - Detection methods:
    - Signature-based (known attacks)
    - Heuristic (suspicious patterns)
    - Behavioral (abnormal activity)
    - IP reputation / location filtering

<img width="1352" height="882" alt="image" src="https://github.com/user-attachments/assets/8e17c80d-a894-4b72-b4b3-9b9a7fa437a8" />

<img width="1504" height="609" alt="image" src="https://github.com/user-attachments/assets/009df791-fa45-4259-902a-384111005e43" />

* Antivirus (AV)
  - Protects endpoints (PCs, servers)
  - Detects known malware using signatures
  - Helps catch:
    - Malicious uploads (e.g., web shells)
    - Post-exploitation tools

Key Idea
  - Security is layered (defense-in-depth)
    - CDN protects from outside
    - WAF filters web traffic
    - AV protects the host
   
# 2. Detecting Web Attacks 
## a. Introduction 
  - Web attacks are a common entry point into systems because web apps are publicly acessible and connected to critical backend systems
  - Objectives
    - Learn common client-side and server-side attacks
    - Understand log-based detection (advantages and limitations)
    - Explore network traffic–based detection
    - Learn how and why Web Application Firewalls (WAFs) are used
    - Practice identifying web attacks
   
  - Prerequisites
    - Basic understanding of web attack types (e.g., OWASP Top 10)
    - Familiarity with log analysis
    - Basic knowledge of packet analysis (e.g., Wireshark)

  - Key Idea
    - Detecting web attacks requires combining:
      - Logs
      - Network traffic analysis
      - Security tools like WAF

## b. Client-Side Attacks
<img width="711" height="216" alt="image" src="https://github.com/user-attachments/assets/d048ac22-abec-439b-8a45-5801bb6736f9" />
  - Client-side attacks target the user’s browser or behavior
  - They exploit:
    - Browser vulnerabilities
    - User actions (tricking users)
    - Third-party plugins (increase attack surface)

  - Key Risk
    - Attack happens on the user’s device, not the server
    - Attackers can:
      - Steal cookies/session
      - Impersonate users
      - Access accounts

  - SOC Limitation
    - SOC tools (logs, network traffic) have little visibility
    - Many attacks don’t generate detectable traffic
    - Hard to detect without endpoint/browser security

  - Common Client-Side Attacks
    - Cross-Site Scripting (XSS) → inject & run malicious scripts in browser
    - Cross-Site Request Forgery (CSRF) → trick user into sending requests
    - Clickjacking → trick user into clicking hidden elements

## c. Server-side Attacks
<img width="567" height="256" alt="image" src="https://github.com/user-attachments/assets/eaa5afeb-ae2d-45b4-a19f-1b946d7ce069" />
  - Server-side attacks target vulnerabilities in the web server, application code, or backend systems.
  - Unlike client-side attacks, they attack the system itself, not the user.
  - They are easier for SOC analysts to detect because they leave traces in logs and network traffic.

Common types:
  - Brute-force → tries many passwords to break into accounts
  - SQL Injection (SQLi) → manipulates database queries to steal data
  - Command Injection → executes malicious system commands on the server

-> Key idea:
     - Server-side attacks exploit how the server processes input and can lead to data theft, unauthorized access, or system compromise.

## d. Log-Based Detectionn
  - Web server logs (access & error logs) record every request made to a website.
  - These logs are important for detecting attacks because they contain evidence of user activity.
  - Security analysts can analyze logs to identify patterns like:
    - Scanning activity
    - Exploitation attempts
    - Suspicious or malicious behavior
  - By understanding log formats, analysts can trace and investigate real attack sequences.

  --> Key idea: Logs = evidence → use them to detect and investigate web attacks.

--> Access Log Format
<img width="704" height="912" alt="image" src="https://github.com/user-attachments/assets/59b7557f-649d-4860-a0cd-bce0135f59aa" />
  - Web server logs help track attacker behavior step by step.
  - A typical attack chain in logs:
    - Directory fuzzing → finds valid pages (HTTP 200)
    - Brute-force attack → many repeated POST requests
    - SQL Injection (SQLi) → injects payloads to access data
  - Limitations of logs:
    - Do not show POST body (e.g., passwords, full payloads)
    - Analysts must infer attacks from patterns and status codes
<img width="664" height="310" alt="image" src="https://github.com/user-attachments/assets/86b68153-ba08-4f0c-8d5c-dff2d0223ed9" />

## e.Network-based Detection 
  - Network traffic analysis shows full request/response data (headers, POST body, cookies, files), unlike logs.
  - Helps analysts see exact attack details, such as:
    - Passwords used in brute-force attacks
    - Full SQL injection payloads
    - Data returned from the server (e.g., database dumps)
  - Limitation: Encrypted traffic (HTTPS, SSH) hides payload unless decrypted.
  - Tools like Wireshark allow filtering (e.g., http) and reconstructing sessions with Follow HTTP Stream.

-->  Key idea:
Logs show that something happened → Network traffic shows exactly what happened.

## f. Wen Application Firewall
<img width="546" height="151" alt="image" src="https://github.com/user-attachments/assets/138d9d2b-7f84-4f6c-8d19-4064d902a6f2" />
  - WAF (Web Application Firewall) protects web apps by inspecting and filtering web requests before they reach the server.
  - It uses rules to allow or block traffic based on patterns, IP reputation, behavior, or custom logic.
  - Common rule types:
    - Block known attack patterns (e.g., SQLi tools like sqlmap)
    - Deny malicious IPs
    - Custom rules for specific apps
    - Rate-limiting to stop abuse (e.g., brute-force)
  - WAFs can also use challenge-response (like CAPTCHA) instead of blocking to verify real users.
  - They integrate threat intelligence to automatically block known attackers and stay updated against new threats.
--> Key idea:
WAF = smart filter in front of your website that detects and stops attacks in real time.
<img width="444" height="167" alt="image" src="https://github.com/user-attachments/assets/cab2c4b7-becb-4785-85b6-fac1f236ebd9" />

