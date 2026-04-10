# Web Application Penetration Test of a Healthcare Platform


## Case Summary
- **Objective:** To conduct a comprehensive penetration test of the "Zero Health" web application, a healthcare platform with an integrated AI chatbot. My goal was to identify and exploit security vulnerabilities across its authentication, API, file handling, and AI functions to assess its overall security posture.
- **Scope:** The engagement was a black-box test targeting the live web application (`zero-health.fezzant.com`). The scope included all application features, from user registration and profile management to file uploads and AI chatbot interactions.
- **Tools Used:** Nmap, Burp Suite, jwt.io, curl, and various OSINT tools for reconnaissance.
- **Outcome:** I identified and successfully exploited **11 critical and high-severity vulnerabilities**, including flaws that led to complete administrative takeover, unauthorized access to private patient medical records, and arbitrary code execution. I provided a detailed report with a prioritized remediation plan to address the systemic failures in the application's security controls.


## Tools & Environment
| Tool | Purpose |
| :--- | :--- |
| **Nmap** | Initial reconnaissance to perform port scanning, service version detection, and OS fingerprinting. |
| **Burp Suite Repeater** | The primary tool for intercepting, modifying, and replaying API requests to test for vulnerabilities like BOLA/IDOR and Mass Assignment. |
| **jwt.io & base64.org** | Used to decode, modify, and re-encode JSON Web Tokens to perform the JWT Algorithm Confusion attack. |
| **curl** | Used to send crafted API requests from the command line to test forged tokens and other exploits. |
| **Kali Linux VM** | The primary offensive security platform for running all reconnaissance and exploitation tools. |


## Case Background
I was tasked with performing a professional penetration test for Securyth Consulting against a target application called "Zero Health." The application is a modern healthcare portal that manages sensitive patient data and uses an AI chatbot to assist users. My mission was to adopt the mindset of a real-world attacker and systematically probe the application for any weakness that could compromise the confidentiality, integrity, or availability of patient data.


## Methodology
My testing followed the industry-standard Cyber Kill Chain methodology to ensure a structured and comprehensive assessment, from initial reconnaissance to final objectives.

1.  **Reconnaissance:** I began with passive reconnaissance using tools like Domain Dossier and BuiltWith to map the target's infrastructure and technology stack. I then moved to active reconnaissance with **Nmap** to identify open ports, enumerate services, and fingerprint the operating system.
2.  **Weaponization:** Based on the reconnaissance findings, I formulated attack hypotheses and crafted specific payloads. This included creating a forged JWT with the `alg` header set to `none`, preparing a malicious SVG file for XSS testing, and designing specific prompts to test for AI injection vulnerabilities.
3.  **Delivery:** I delivered the payloads using the application's intended interfaces. For example, the forged JWT was sent via `curl` to a restricted API endpoint, the malicious SVG was submitted through the file upload feature, and the AI prompts were typed directly into the chatbot.
4.  **Exploitation:** I executed the attacks and documented the results. This involved using **Burp Suite Repeater** to incrementally change patient IDs to test for IDOR and sending modified JSON objects to test for Mass Assignment.
5.  **Actions on Objectives:** After gaining privileged access through exploits like JWT forgery and SQL injection, I confirmed the total impact. I successfully accessed administrative functions, viewed private medical records of other patients, and executed arbitrary commands, achieving the primary objectives of a real-world attacker.


## Key Findings & Vulnerabilities
The test uncovered systemic failures in core security controls. The top three most critical vulnerabilities are detailed below.

| Vulnerability | CVSS Score | Finding & Impact |
| :--- | :--- | :--- |
| **JWT Algorithm Confusion (None-Algorithm Attack)** | **9.8 (Critical)** | The server failed to validate the JWT signature, allowing me to forge a token with an 'admin' role. This granted me immediate, full administrative access to the entire application. |
| **SQL Injection (Authentication Bypass)** | **9.8 (Critical)** | The login endpoint was vulnerable to SQL injection. By sending a payload of `' OR '1'='1' --`, I bypassed the password check and logged in as the first user in the database, who happened to be the administrator. |
| **Agentic Privilege Escalation via AI Chatbot** | **9.1 (Critical)** | The AI chatbot was directly connected to backend administrative functions without its own permission checks. As a low-privilege user, I could simply type "delete user test456@gmail.com" and the AI would execute the command, demonstrating a complete breakdown of function-level authorization. |


## Conclusion
The penetration test identified multiple, chained critical security weaknesses that placed the Zero Health application and its sensitive patient data at extreme risk. An attacker could easily gain administrative control, access any patient's private medical records, execute malicious code in other users' browsers, and cause severe disruption.

**Impact:** The combination of these vulnerabilities could lead to catastrophic consequences in a healthcare environment, including mass disclosure of patient records (leading to regulatory fines and lawsuits), administrative takeover, and reputational collapse.

**Recommendations:** My final report strongly recommended immediate remediation, with the highest priority on:
1.  **Fixing Authentication:** Enforce strict server-side validation of all JWT signatures and use parameterized queries to prevent all forms of SQL injection.
2.  **Enforcing Authorization:** Implement robust, server-side authorization checks on every single API request to prevent IDOR and Mass Assignment. The AI chatbot must inherit and respect the permissions of the user it is interacting with.
3.  **Hardening Inputs and Outputs:** Implement strict file type validation on uploads, properly sanitize all API responses to prevent data leakage, and return generic error messages to prevent fingerprinting.


## Lessons Learned / Reflection
This engagement was a powerful case study in how modern application vulnerabilities often chain together to create a devastating impact. The JWT flaw gave me the keys, but the IDOR vulnerability opened all the doors. This project drove home the lesson that **security cannot be a frontend-only concern.** Many of the most critical flaws, like the AI's broken authorization and the Mass Assignment vulnerability, were invisible from the user interface and could only be found by directly interrogating the API. It also highlighted the new attack surface introduced by AI; a chatbot is not just a text generator but a potential new interface to backend functions that must be secured with the same rigor as any other API endpoint.
