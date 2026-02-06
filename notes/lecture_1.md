## Class Notes — Cloud Computing (AWS EC2), Security, Regions/AZs, and Access (SSH/RDP)

### 1) Core story / why this matters (real incident)

* Cloud accounts can be **abused fast** if security is weak (no MFA, open access, unused resources left running).
* Example discussed: someone gained access to an account, spun up many compute instances, and ran **unauthorized workloads** (often crypto-mining). The bill can explode (e.g., “hundreds of thousands”).
* Key takeaway: **MFA + least privilege + cleanup** are not “optional.” They prevent real money loss.

---

## 2) Key Concepts (with verified definitions)

### A) Region (Where your cloud resources live)

* A **Region** is a **geographic area** (e.g., US East (Ohio), US West (Oregon)). ([AWS Documentation][1])
* You choose a Region when creating services (EC2, databases, etc.).

**Verified examples mentioned in lecture**

* **us-east-1 = US East (N. Virginia)** ([AWS Documentation][1])
* **us-east-2 = US East (Ohio)** ([AWS Documentation][1])
* **us-west-1 = US West (N. California)** ([AWS Documentation][1])
* **us-west-2 = US West (Oregon)** ([AWS Documentation][1])

**Correction / verification note**

* The lecture said “there are four regions” (in that context). AWS actually has **more than four** US Regions today; the “four” is likely just the common set being discussed for class examples. ([AWS Documentation][1])

---

### B) Availability Zone (AZ) (Reliability units inside a Region)

* An **Availability Zone** is **one or more discrete data centers** within a Region, with **separate and redundant power, networking, and connectivity**. ([AWS Documentation][2])
* AZs are named like `us-east-2a`, `us-east-2b`, etc. ([AWS Documentation][3])

**Why it matters**

* Multi-AZ designs improve **high availability**: if one AZ fails, your service can keep running in another AZ.

---

### C) EC2 instance (Virtual machine)

* EC2 = AWS virtual server you “rent” on demand.
* You select:

  * **Region + AZ**
  * **Hardware shape** (“instance type”: CPU/RAM/network)
  * **AMI** (software image: OS + sometimes preinstalled packages)
  * **Security controls** (key pair + security group)

---

## 3) Decision checklist: choosing a Region (lecture points)

**Main factors**

1. **Latency / customer proximity** (closer region → faster response)
2. **Legal / compliance constraints** (data residency rules: “data must stay in EU,” etc.)
3. **Service availability** (some services/features aren’t in every region)
4. **Cost** (pricing differs by region)

**Visualization**

```
Choose Region
   |
   +-- Is user/customer mostly in a specific area? --> pick closest
   |
   +-- Any legal/data residency rule? --> must stay in allowed regions
   |
   +-- Need specific service (AI/ML/GPU/etc.)? --> verify region supports it
   |
   +-- Compare pricing --> choose cost-effective option
```

---

## 4) Instance Types (hardware families) — what the lecture meant

* AWS groups instance types by purpose:

  * **General purpose** (balanced CPU/RAM) — common for web servers
  * **Compute optimized** — heavy CPU workloads
  * **Memory optimized** — heavy RAM workloads
  * **Storage optimized** — heavy disk I/O workloads

Lecture decoding example:

* `t3.micro`:

  * `t` family
  * `3` generation
  * `micro` size (small)

---

## 5) AMI (Amazon Machine Image) — software choice

* AMI = the “image” used to launch an instance (OS + base config).
* Lecture used:

  * **Amazon Linux** AMI for Linux EC2
  * **Windows Server** AMI for Windows EC2

---

## 6) Two lines of defense (security) — MUST KNOW

### Line 1: MFA (Account-level protection)

* MFA helps prevent account takeover even if password is leaked.
* Lecture advice: enable MFA early, because a stolen account can be abused globally.

### Line 2: Key Pair (Instance login identity)

* **Public key** is associated with the instance.
* **Private key** stays with you.
* If you lose the private key, you may lose normal access to the instance (you often must rebuild/replace access path).

### Line 3: Security Group (Network firewall)

* Security group = **virtual firewall** controlling:

  1. **What traffic** (protocol/port like SSH/HTTP/RDP)
  2. **From where** (source IP/CIDR)

**Important “verify” note (from lecture Q&A)**

* “My IP” source rules can break if your external IP changes (Wi-Fi changes, restart, ISP changes).
* In many networks, your apparent “public IP” is actually the **NAT egress IP** shared by many users; that’s why multiple people can appear to come from the same public IP.

---

## 7) Ports & protocols (exam-style)

* **SSH** (Linux remote login): **port 22**
* **HTTP** (web): **port 80**
* **HTTPS** (secure web): **port 443**
* **RDP** (Windows remote desktop): commonly **port 3389**

**Visualization**

```
Internet user --> [Security Group] --> EC2 instance
                  allow 22? (SSH)
                  allow 80? (HTTP)
                  allow 443? (HTTPS)
                  allow 3389? (RDP)
```

---

## 8) Access methods shown in lecture

### A) Linux EC2 access (SSH)

**Windows path (GUI)**

* Convert `.pem` → `.ppk` using **PuTTYgen**, then login using **PuTTY**. ([puttygen.com][4])

**Mac/Linux path (CLI)**

* Use `ssh -i yourkey.pem ec2-user@<public-ip>`
* Default username depends on AMI; for Amazon Linux it’s commonly `ec2-user` (as shown in lecture).

### B) Windows EC2 access (RDP)

* Use **Remote Desktop** to public IP.
* Get the **Administrator password** by decrypting it using your **private key** (AWS console workflow), then RDP in.

---

## 9) Availability Zone & High Availability visualization (what the professor explained)

```
Region: us-east-2 (Ohio)
  AZ-a         AZ-b         AZ-c
 [EC2-1]      [EC2-2]      [EC2-3]
     \           |           /
      \          |          /
       +---- [Load Balancer] ----+
                 |
              Users
```

If **AZ-a** fails, traffic can still go to **AZ-b / AZ-c**.

---

# Practice Questions (from this lecture)

### Quick checks (1–2 lines)

1. What is the difference between a **Region** and an **Availability Zone**?
2. What two things does a **Security Group** rule specify?
3. Why is “**0.0.0.0/0** on SSH (22)” dangerous?
4. Why should you **delete resources after labs**?

### Scenario questions

5. You’re building a website for UTD users. List **4 region-selection factors** and pick a region.
6. You launched an EC2 but can’t access your website. What security group ports might you be missing?
7. You limited SSH to “My IP,” but now you can’t connect from home. Name 2 likely causes and 2 fixes.

### Deeper reasoning

8. Why do Multi-AZ designs improve availability? Tie it to failure isolation. ([AWS Documentation][2])
9. Why is MFA considered essential for cloud accounts (use the lecture incident)?

---

## “Verified vs lecture” — short audit

* ✅ Region codes and locations (us-east-1, us-east-2, us-west-1, us-west-2) are correct. ([AWS Documentation][1])
* ✅ AZ definition (separate/redundant power + networking + connectivity) is correct. ([AWS Documentation][2])
* ✅ AZ naming format (region + letter, e.g., us-east-2a) is correct. ([AWS Documentation][3])
* ✅ PEM→PPK conversion via PuTTYgen for PuTTY is a standard workflow. ([puttygen.com][4])
* ⚠️ “Only four regions” — not true globally; likely a simplified class subset. ([AWS Documentation][1])

---

If you want, I can turn this into a **1-page exam cheat sheet** (same content, tighter) + a **mini quiz** (10 MCQ + 5 short answers) based only on this lecture.

[1]: https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions.html?utm_source=chatgpt.com "AWS Regions and Availability Zones"
[2]: https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/availability-zones.html?utm_source=chatgpt.com "Availability Zones - AWS Fault Isolation Boundaries"
[3]: https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-availability-zones.html?utm_source=chatgpt.com "AWS Regions and Availability Zones"
[4]: https://www.puttygen.com/convert-pem-to-ppk?utm_source=chatgpt.com "Convert Pem to Ppk File Using PuTTYgen"
