# 🛡️ Enterprise Phishing Simulation & Defense Lab (AWS)

## 📌 Project Summary
This project demonstrates the end-to-end deployment of a phishing simulation environment hosted on **AWS EC2**. It combines secure mail infrastructure, phishing campaign delivery, DNS authentication, and defensive analysis in a controlled lab environment.

The lab was built to replicate realistic security engineering challenges, including DNS configuration, SMTP deliverability, container orchestration, and infrastructure troubleshooting. The goal was not only to simulate phishing activity, but also to understand how the supporting mail stack works, how to validate it, and how defenders can use those same artifacts for detection and awareness training.

---

## 📌 Project Overview
This project demonstrates the end-to-end deployment of a professional phishing simulation environment hosted on **AWS**. Beyond simply running a "phish," this lab focuses on the **Security Engineering** required to build a reliable, authenticated mail infrastructure and the **Defensive Analysis** used to educate end-users.

### 🏗️ Architecture & Stack
* **Cloud Infrastructure:** AWS EC2 (c7i-flex.large)
* **Orchestration:** [Gophish](https://getgophish.com/) (Open-Source Phishing Framework)
* **Mail Server:** [Poste.io](https://poste.io/) (Dockerized SMTP/IMAP Stack)
* **DNS Management:** Porkbun (SPF, DKIM, DMARC)
* **Target Domain:** `harvenorman.click`

---

## 🛠️ Deployment Phases

### 1. Cloud Infrastructure Provisioning
* **Instance Scaling:** Initially deployed on `t3.micro`. Due to resource contention with the Poste.io Antivirus/Spam engines, the environment was migrated to a **c7i-flex.large** (2 vCPU, 4GB RAM) for stability.
* **Elastic IP:** Assigned a static IPv4 to ensure DNS record consistency.
* **Security Groups:** Configured granular Inbound/Outbound rules to allow:
    * `SSH (22)` - Management
    * `HTTP/S (80, 443)` - Landing Pages & Admin UI
    * `SMTP Submission (587)` & `SMTPS (465)` - Mail Delivery

### 2. DNS & Mail Authentication (The "Blue Team" Layer)
To ensure high deliverability and simulate real-world attacker sophistication, the following records were implemented:
* **SPF (Sender Policy Framework):** `v=spf1 ip4:[REDACTED_IP] -all` (Hard Fail enforced).
* **DKIM (DomainKeys Identified Mail):** 2048-bit RSA key implemented via the `mail` selector.
* **DMARC:** Configured to `p=none` for initial monitoring, showing an understanding of the phased rollout of mail security policies.

### 3. Campaign Design
* **Email Template:** A "Security Alert" mimicking a Corporate SOC notification, utilizing a "Sense of Urgency" social engineering tactic.
* **Landing Page:** A custom-built HTML/CSS page that intercepts clicks and redirects users to a **Security Awareness Training** portal rather than harvesting credentials (Ethical Compliance).

---

## 🔍 Troubleshooting & Engineering Challenges
*A key part of this lab involved overcoming real-world infrastructure hurdles.*

| Challenge | Root Cause | Resolution |
| :--- | :--- | :--- |
| **Instance Freezing** | `t3.micro` CPU/RAM exhaustion by Docker containers. | Upgraded to **c7i-flex.large**; implemented swap space for stability. |
| **AWS Port 25 Block** | AWS default egress filtering to prevent spam. | Pivoted to **authenticated SMTP over Port 587** and **SMTPS on 465**. |
| **DNS Resolution Error** | `lookup mail.harvenorman.click: no such host` | Diagnosed missing **A Record** for the `mail` subdomain; verified via `ping` and `nc`. |
| **Connection Refused** | Security Group missing Port 80/443 mapping. | Audited AWS Inbound Rules and mapped Docker ports `80:80` and `443:443`. |
| **Webmail Unreachable** | Browser could not reach `/webmail` using the domain before DNS and security rules were fully aligned. | Verified access using the EC2 public IP first, then confirmed domain-based access after DNS propagation and inbound rule validation. |
| **SMTP Relay Errors in Gophish** | Gophish could not resolve the mail server hostname from inside the container. | Switched the Sending Profile SMTP host to `localhost:587`, which allowed the container to reach Poste.io successfully. |
| **Outbound Delivery to Yahoo Failed** | Yahoo filtered or rejected messages from the new lab IP/domain. | Confirmed this as a deliverability/reputation issue and used Gmail for primary testing. |
| **EC2 SSH Access Error** | `Permission denied (publickey)` while connecting with the `.pem` key. | Verified the key file location and corrected permissions with `chmod 400 my-ec2-key.pem`. |
| **GoPhish Archive Confusion** | `cd gophish-v0.12.1-linux-64bit` failed because unzip extracted files into the current directory. | Ran `./gophish` directly from the extracted folder instead of changing into a non-existent subdirectory. |
| **GoPhish Port Bind Error** | `listen tcp 0.0.0.0:80: bind: permission denied`. | Updated `config.json` to use port `3333` for the admin panel. |
| **GoPhish Username Lookup** | Needed to identify the default/admin username stored in the SQLite database. | Queried the database using `sqlite3 gophish.db` and `SELECT id, username FROM users;`. |
| **Poste.io Docker Deployment** | Needed a persistent, containerized mail server with exposed ports for SMTP and webmail. | Pulled `analogic/poste.io`, created a persistent data directory, and ran the container with mapped ports and a volume mount. |
| **SMTP Timeout from Local Machine** | Telnet from the local machine to ports `587` and `465` timed out. | Confirmed the container was listening correctly by testing from the EC2 host first, then added AWS Security Group rules and OS firewall rules. |
| **Local Host Validation** | Needed to verify whether the SMTP service worked on the EC2 host itself. | Used `telnet 127.0.0.1 587` and `telnet 127.0.0.1 465`, confirming the service responded correctly. |
| **AWS Security Group Blocking SMTP** | SMTP ports were not reachable from outside the instance. | Added inbound rules for TCP `587` and `465` from my IP. |
| **Ubuntu Firewall Blocking SMTP** | OS-level firewall restricted access to the mail ports. | Allowed the ports with `ufw allow 587/tcp` and `ufw allow 465/tcp`, then reloaded UFW. |
| **DNS Not Found in GoPhish Test Email** | `mail.mydomain` was not yet resolvable during SMTP testing. | Added the correct mail hostname at the registrar and verified propagation with `nslookup`. Temporarily used the EC2 IP in the GoPhish sender profile until DNS fully propagated. |

---

## 📊 Results & Artifacts
> **[Optional: Insert Screenshot of Gophish Dashboard here]**
> *Caption: Real-time tracking showing successful Email Sent, Opened, and Clicked events.*

> **[Optional: Insert Screenshot of Mail Headers here]**
> *Caption: Proving `PASS` status for SPF and DKIM signatures.*

> **[Optional: Insert Screenshot of Poste.io Diagnostics here]**
> *Caption: Demonstrates DNS and mail service validation after infrastructure troubleshooting.*

> **[Optional: Insert Screenshot of Porkbun DNS Records here]**
> *Caption: Shows A, MX, SPF, DKIM, and DMARC configuration for the mail domain.*

> **[Optional: Insert Screenshot of intodns Report here]**
> *Caption: Displays external DNS validation results and highlights common mail setup warnings.*

---

## 🧠 Lessons Learned
1. **Infrastructure as Code (Thinking):** The importance of selecting the right compute class for containerized security tools.
2. **Mail Integrity:** How modern mail filters use SPF/DKIM to build trust, and how attackers leverage these same tools.
3. **Remediation:** The simulation doesn't end at the "click"—the goal is the **Teachable Moment** provided by the landing page redirect.
4. **Deliverability Reality:** New mail domains and IPs may work technically but still face filtering from major providers like Yahoo and Gmail.
5. **Container Networking:** Understanding how Docker networking, hostname resolution, and host-based services interact was critical to getting Gophish and Poste.io to work together.
6. **Operational Troubleshooting:** The lab reinforced the value of testing incrementally, starting with DNS, then web access, then SMTP, then campaign delivery.
7. **SSH Hardening Awareness:** Correct `.pem` permissions are required for AWS EC2 authentication.
8. **Linux Service Validation:** Confirming a service on `127.0.0.1` first is a reliable way to isolate host vs. network issues.
9. **Telnet Workflow:** `Ctrl + ]` followed by `quit` is useful for exiting stuck telnet sessions.
10. **Port Planning:** Running multiple network services on one instance requires careful mapping of host ports and Docker container ports.

---

## 👨‍💻 How to Use
1. Clone this repo.
2. Deploy the AWS CloudFormation template (if applicable).
3. Install Docker and run the provided `docker-compose.yml`.
4. Configure your Gophish Sending Profile to point to your Elastic IP.
5. Validate DNS records, SMTP submission, and webmail access before launching any campaign tests.

---

## 📁 Suggested Repository Structure
```text
.
├── README.md
├── screenshots/
├── docker/
├── configs/
├── notes/
└── diagrams/
```

---

## 📌 Screenshots
- AWS EC2 instance overview
- Security group inbound rules
- Docker container setup
- Poste.io dashboard
- Poste.io user creation page
- Webmail inbox
- DNS records in Porkbun
- intodns health check
- Poste.io diagnostics
- GoPhish admin panel
- GoPhish sending profile
- Test email failure and troubleshooting steps
- Working SMTP submission after fixing the host value

---

## 📚 References
- [GoPhish](https://getgophish.com/)
- [Poste.io](https://poste.io/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Porkbun DNS Documentation](https://kb.porkbun.com/)

---

*Created by [Your Name] | Cybersecurity Analyst & Cloud Security Enthusiast*
