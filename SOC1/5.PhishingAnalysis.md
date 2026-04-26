# 1. Fundamentals
## a. Introduction 
- Spam and phishing are the most common social engineering threats. While spam is usually harmless, phishing is dangerous because it can trick users into revealing sensitive information or installing malvware. A single careless click can allow attackers to access a network and cause serious damage.

- Defenders must analyze emails to determine whether they are malicious or safe. This involves examining email headers, content, and attachments to identify suspicious indicators and understand the true source of the message.

- Learning objectives include:
    - Understanding how email delivery works
    - Analyzing email headers
    - Investigating email content (body)
    - Finding different types of phishing attacks
    - Detecing potential security threats in emails

  ## b. The Email Address
  An email address consists of three main parts
  - Username : identifies the recipient's mailbox
  - @symbol : separate the username and domain
  - Domain name : indicates the mail server that handles the email
    It works like a home address : the domain is like the location (street/building), and the username is the specific person. Understanding this structure helps detect phishing emails by spotting unusual or fake domains
    Example : 
  <img width="1801" height="608" alt="image" src="https://github.com/user-attachments/assets/5dd59440-035c-4872-b28d-c33a784464a5" />

  ## c. Email Delivery
  Email delivery relies on three main protocols working together
  - SMTP is used to send emmails from the sender to a mail server
  - POP3 downloads emails to a single device and usually removes them from the server
  - IMAP keeps emails on the server and syncs them across multiple devices
*  The journey of an email involves sending via SMTP, using DNS to find the recipient's mail server, delivering the message, and then retriving it via POP3 or IMAP. IMAP is better for accessing email on multiple devices, while POP3 is suited for single-device access.

## d. Email Headers
An email has two main parts:
  - Header : contains metadata such as sender, receiver, subject, date, and routing details
  - Body : contains the actual message (text or HTML)
Email headers are very important in security analysis because they reveal hidden technical details like the sender's real origin and the path the email took. By viewing the raw message source (e.g : in Thunderbird using View --> Message Source or CTRL + U), analysis can see full header information, including fields like X-Originating-IP, which helps identify where the email actually came from.

## e. Email Body
The email body contains the actual message and can be either plain text or HTML. HTML emails may include links, images, and styling, which can hide malicious content. By viewing the HTML source, analysts can inspect hidden elements and detect phishing indicators such as suspicious links or embedded resources

Email can also include attachments, which are encoded (often in Base64) within the message source. Important headers for attachments include:
  - Content-Type : file type (e.g: application/pdf)
  - Content-Disposition: indicates it's an attachment and shows the filename
  - Content-Transfer-Encoding : usually base64
By extracting and decoding the Base64 data, analysts can reconstruct the original file and uncover hidden content like flags or malware

## f. Types of Phishing 
Attackers often exploit email systems using social engineering to trick users. Common malicious email types include spam, phishing, spear phishing, whaling, smishing, and vishing

Phishing emails usually share recognize traits:
- Fake sender addresses (spoofing trusted organizations)
- Urgent or threatening messages
- Brand impersonation (logos, design)
- Generic greetings
- Suspicious links or attachments

* For safe analysis, links and IPs should be defanged (e.g., replacing . with [.]) to prevent accidental clicks. By examining headers (like X-Originating-IP and Authentication-Results), defenders can uncover the real source of the email.

# 2. Phishing  Emails in action 
## a. Introduction
This task marks the transition from theory to practice in phishing analysis. You will be analyze real phishing email samples to understand how attackers mimic legitmate communications. The goals is to identify subtle signs that distinguish normal emails from malicious ones

Focus on: 
  - Recognizing social engineering tactics
  - Identifying red flags in emails
  - Detecting malicious links and tracking pixels
  - Understanding credential harvesting and malicious attachments
* ** Important** : These are real sammples, so avoid interacting links, IPs, or attachments

## b. Cancel your order
This phishing email pretends to be a Paypal transaction receipt to trick the victim into reacting quickly. Attackers use serveral techniques:
  - Spoofed email address: Looks like PayPal but actually from a different domain
  - Urgent subject: Fake purchase to create panic
  - Branded HTML: Designed to look like a real PayPal email
  - URL shortening: Hides the real malicious destination behind a shortened link
--> The email body shows a fake purchase (gift cards) and includes a “Cancel the order” button, which leads to a hidden malicious link. Always verify links before clicking.

## c. Track Your Package
This phishing email pretends to be a shipping notifications to create urgency. Attackers use: 
  - Spoofed sender: Fake "Distribution Center" but real email is suspicious
  - Tracking pixels: Invisible images to detect when the email is opened
  - Link manipulation : A fake tracking number that hides the real destination
Email providers like Yahoo block images and links because they may contain tracking or malicious content. By inspecting the raw source, analysts can uncover the true destination behind hidden links

* First Observations:
  - Subject line : USes a fake tracking number to create urgency --> pushes user to click
  - From address : Display name "Distribution Center" ≠ real email contact@beginpro.club
 → suspicious
  - To address: May look unusual or not personalized --> another red flag

* Hyperlink :
  - The tracking number appears clickable but is actually masked
  - The real destination is hidden behind HTML or an image (tracking pixel)
  - You must check the raw source to see the real link
  - The link points to a suspicious domain → beginpro[.]club
 
## d. Download Document Here
The phishing campaign uses a multi-stage redirection chain to trick users into giving away their login credentials. It impersonates trusted services like Microsoft, OneDrive, and Adobe to build credibility.

Key techniques include:
  - Artificial urgency (same-day expiration)
  - Brand impersonation (Microsoft, Adobe)
  - Link redirection (multiple steps to hide final destination)
  - Fake login page to steal credentials
Even if users enter correct credentials, the page simply captures and sends them to the attacker instead of authenticating

## e. Your Account Is on Hold
This phishing email pretends to be a Netflix billing alert to pressure the user into taking immediate action. Instead of using a direct malicious link, it includes a PDF attachment that contains the harmful link.

* Key red flags:
  - Spoofed sender (“Netllx billing” instead of Netflix)
  - Urgency (account suspended)
  - Brand impersonation (fake Netflix design)
  - Typos (“Netllx”)
  - Malicious attachment (PDF with hidden link)
  - Suspicious phone number and mismatched domains
  --> The goal is to trick the user into opening the attachment and clicking the embedded link.

## f. Your Recent Purchase
This phishing email impersonates Apple Support and uses a blank body with a suspicious attachment to trick the user. It creates urgency with a fake purchase alert and hides recipients using BCC. The attachment is a .dot file (Word template) that contains a malicious link disguised as legitimate.

* Key red flags:
  - Spoofed sender (Apple Support)
  - BCC used to hide recipients
  - Urgent fake purchase notification
  - Blank email body
  - Suspicious attachment format (.dot)
  - Malicious redirect link inside the file
 
## g. Scheduled Shipment
This phishing email impersonates DHL Express and uses a malicious Excel attachment to execute harmful code. The email appears legitimate with branded HTML, but contains inconsistencies (different countries/languages), which are clear red flags.

The Excel file includes a malicious hyperlink that downloads an executable. This file is intended to run on the victim’s machine and could allow attackers to gain access, steal data, or deploy ransomware.

