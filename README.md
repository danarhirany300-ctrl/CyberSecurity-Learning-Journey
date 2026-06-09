# 🛡️ SOC Analyst Learning Journey

Welcome to my daily documentation tracker as I build hands-on skills for a Security Operations Center (SOC) Analyst role. 

---

## 📅 Day 1: Network Security Foundations

### 🧠 Core Concept: What is a Network & Data Movement
A network is fundamentally just computers talking to each other. It is not an abstract "cloud"—it consists of physical assets: your device, a local router, an Internet Service Provider (ISP), and destination servers.

* **The Packet:** Data does not travel all at once; it is broken down into small pieces called **packets** (like an envelope).
* **Packet Structure:** Contains a **Source IP** (where it comes from), a **Destination IP** (where it is going), and the **Payload** (the actual data/request).
* **The Security Perspective:** *Every network attack is just modified, suspicious, fake, or stolen packets.* Understanding normal packet flow is the prerequisite to identifying malicious packet behavior.

---

### 📦 The Real Path of a Packet (Routing)
When requesting data from a server (e.g., watching a video on YouTube), data moves through a multi-step delivery system:

1.  **Device (Start Point):** Generates the request and packages it into a packet.
2.  **Router (Wi-Fi Box):** Acts as the local traffic manager, deciding where to send the packet next.
3.  **ISP (Internet Service Provider):** Connects the local home network to the global internet infrastructure.
4.  **Internet Routers (Hops):** The packet moves from router to router across the globe via a process called **Routing**.
5.  **Destination Server:** The website receives the packet, processes the request, and sends response packets back along the reverse path.

> ⚠️ **Security Engineering Insight:** Understanding this path allows analysts to detect the origin of attacks, identify where traffic is intercepted, and spot Man-In-The-Middle (MITM) or packet-sniffing exploits.

---

### 🏗️ The OSI Model: Mapping the Network
The OSI Model is a 7-layer map that defines where specific network actions—and security threats—occur. 

| Layer | Name | Core Function | Security Context |
| :--- | :--- | :--- | :--- |
| **Layer 7** | Application | What the user interacts with (Browsers, Apps) | Web exploits, login forms, SQL Injection, XSS |
| **Layer 4** | Transport | Controls delivery rules (**TCP** / **UDP**) | Port scanning, managing open communication ports |
| **Layer 3** | Network | Handles routing and **IP addresses** | IP spoofing, routing attacks |
| **Layer 2** | Data Link | Local network communication (**MAC addresses**) | Local sniffing, ARP poisoning attacks |
| **Layer 1** | Physical | Physical signals (Wi-Fi waves, Ethernet cables) | Physical breaches, hardware tampering |

### 🛠️ Day 1 Mental Walkthrough
When connecting to a secure website via Wi-Fi:
* **Browser (Chrome/Firefox)** operates at **Layer 7**.
* **HTTPS (TCP connection)** rules operate at **Layer 4**.
* **IP Routing** across the internet operates at **Layer 3**.
* **Wi-Fi connection** to the home router operates at **Layer 2**.
* **Radio waves** transmitting the data operate at **Layer 1**.

* Add Day 1 Notes



## 📅 Day 2: Network Visibility & Protocols (Wireshark, DNS, & TCP)

### 🔬 Core Tool: Wireshark (Network Traffic Analysis)
Moving from theory to reality requires packet visibility. **Wireshark** acts as a network packet analyzer (effectively a "CCTV camera" for network traffic), capturing and displaying data packets traveling across a network interface in real time.

* **Network Interfaces:** The point where a device connects to a network (e.g., `Wi-Fi` for wireless, `Ethernet` for wired cables, or `Loopback` for internal system testing).
* **The Baseline Noise:** Live networks are constantly generating "noisy" background traffic. Analyzing traffic requires filtering out the noise to focus on specific protocols, source IPs, or anomalous indicators.
* **The Security Perspective:** Wireshark is the fundamental tool for traffic monitoring, intrusion detection, and incident response. Without packet visibility, a SOC analyst is blind to active threats.

---

### 📖 Protocol Focus: DNS (The Internet Phonebook)
Computers do not communicate via human-readable domain names (like `google.com`); they communicate via numerical IP addresses. The **Domain Name System (DNS)** resolves domain names into routable IP addresses.



* **Packet Filtering:** In Wireshark, applying the display filter `dns` isolates domain queries and responses.
* **DNS Record Structure:** A standard query seeks an **A Record** (which maps a hostname to an IPv4 address).
* > ⚠️ **Security Engineering Insight:** DNS traffic often remains unencrypted even if the subsequent web traffic is encrypted via HTTPS/TLS. Attackers frequently target DNS through **DNS Poisoning** or malicious redirection. For an analyst, DNS logs reveal exactly what external domains an internal asset is attempting to contact *before* an encrypted session ever begins.

---

### 🤝 Protocol Focus: The TCP 3-Way Handshake
Before data can reliably move between a client and a server, the **Transmission Control Protocol (TCP)** must establish a connection. This connection is opened using a mandatory, structured process called the **3-Way Handshake**.
Client                               Server
|                                    |
| ------------ SYN ------------->    |  (Can we connect?)
|                                    |
| <---------- SYN-ACK -----------    |  (Yes, I am ready. Are you?)
|                                    |
| ------------ ACK ------------->    |  (Confirmed. Connection open.)
|                                    |


* **Step 1: SYN (Synchronize):** The client sends a packet with the SYN flag set to request a connection.
* **Step 2: SYN-ACK (Synchronize-Acknowledge):** The server responds, confirming receipt of the request and asking to synchronize back.
* **Step 3: ACK (Acknowledge):** The client sends a final confirmation packet. The connection is now established, and data (such as web pages) can safely flow.

#### 🧠 Analyst Thinking: Analyzing the Handshake in Wireshark
By applying the `tcp` filter in Wireshark, we can audit these handshakes to diagnose network health or malicious intent:
* **Normal Behavior:** A clean sequence of `SYN` ➔ `SYN-ACK` ➔ `ACK`.
* **Suspicious Behavior (Reconnaissance):** A flood of rapid `SYN` packets to multiple different ports without ever completing the handshake with an `ACK`. This is a classic **SYN Port Scan** used by attackers to map out open vulnerabilities on a target system.
* **Denial of Service (DoS):** Flooding a server with endless `SYN` requests to exhaust its resources keeping half-open connections waiting.

* Add Day 2 Notes


## 📅 Day 3: Reconnaissance, Local Attacks, & Traffic Monitoring

### 🔌 Protocol Abuse: Port Scanning (Finding the Open Doors)
An IP address identifies a device on a network, but **Ports** identify the specific services or applications running on that device. Think of an IP address as a building's street address, and ports as the individual doors inside.

#### Common Ports to Memorize:
* **Port 22:** SSH (Remote Secure Login)
* **Port 53:** DNS (Domain Name Resolution)
* **Port 80:** HTTP (Unencrypted Web Traffic)
* **Port 443:** HTTPS (Encrypted Web Traffic)
* **Port 3389:** RDP (Remote Desktop Protocol)

**Port Scanning** is the systematic probing of a target system to discover which ports are open. Attackers use tools like **Nmap** to map out these open doors before launching an exploit.



#### 🔍 The Wireshark Detection Pattern:
When a port scan occurs, the traffic pattern is highly predictable:
* A flood of rapid **SYN** packets sent to a wide range of different ports from a single source IP.
* The absence of completion packets (**ACK**). 
* **The Signature:** `SYN` ➔ `SYN` ➔ `SYN` ➔ `SYN` without ever finalizing the 3-way handshake. This indicates an attacker is scanning to see what responds.

---

### 📡 Protocol Abuse: ARP Spoofing (Tricking the Local Network)
The **Address Resolution Protocol (ARP)** is used inside local networks (like a home Wi-Fi or office LAN) to map an abstract IP address to a physical hardware identity, known as a **MAC Address**.

* **The Analogy:** If an IP address is a house address, a MAC address is the specific identity of the person living there. ARP continuously asks: *"Who has this IP? Tell your MAC address to the network."*
* **The Vulnerability:** ARP was designed without built-in authentication. Devices blindly trust and cache incoming ARP replies.

#### 💣 Man-in-the-Middle (MITM) via ARP Poisoning:
An attacker can exploit this lack of authentication by broadcasting fake ARP replies to the local network, claiming that *their* MAC address belongs to the default gateway (the local router).



* **The Result:** The victim device sends all its internet traffic directly to the attacker instead of the router, allowing the attacker to intercept, sniff, or modify data.
* **Detection:** In Wireshark, filtering by `arp` allows an analyst to look for duplicate MAC address entries or a single MAC address suddenly claiming ownership over multiple distinct IP addresses.

---

### 👀 Defensive Mastery: Traffic Monitoring
Traffic monitoring is the continuous, real-time observation of network packets to distinguish **Normal Baseline Behavior** from **Suspicious Anomalies**. 

Security engineers do not look for a magical "hacked" packet; they look for abnormal behavior patterns inside standard protocols.

| Traffic Type | Indicators & Patterns |
| :--- | :--- |
| **Normal Traffic** | Occasional DNS queries, completed TCP handshakes (`SYN` ➔ `SYN-ACK` ➔ `ACK`), steady HTTPS streams. |
| **Suspicious: Port Scan** | High-volume, rapid `SYN` packets targeting multiple sequential ports with no `ACK` follow-up. |
| **Suspicious: ARP Spoofing** | Sudden floods of unsolicited ARP replies mapping a single MAC address to different device IPs. |
| **Suspicious: DNS Anomalies** | Repeated requests for random-looking, long, or unreadable domain names (indicative of malware beaconing). |

> 🧠 **The SOC Analyst Mindset Shift:** A great security engineer shifts their focus from asking *"What does this packet mean?"* to asking *"Is this specific behavioral pattern normal or malicious for my environment?"*


Add Day 3 Notes



## 📅 Day 4: The Full Attack Lifecycle (The Cyber Kill Chain)

### 🔗 Core Concept: The Multi-Stage Attack Path
Real-world cyber attacks are rarely single, isolated events. Instead, they occur as a structured chain of sequential phases over time. Security professionals refer to this progression as the **Attack Lifecycle** or **Attack Path**.



---

### 💣 The 4 Core Stages of a Network Attack

#### 🔍 1. Reconnaissance (Target Discovery)
The attacker maps out the landscape to identify live targets, active IP addresses, and open ports.
* **Primary Tool:** `Nmap` (Port scanning and network mapping).
* **SOC Visibility Indicator:** A high volume of rapid `SYN` packets targeting multiple ports from a singular external source IP without establishing completed connections.

#### 🚪 2. Enumeration (Deep Information Gathering)
Once an open port or service is discovered, the attacker probes deeper to find out exactly *what* is running behind that door.
* **Objective:** Identify exact software versions, operating systems, and misconfigured services.
* **The Danger:** Vulnerabilities are highly specific to software versions. Knowing the exact version allows attackers to look up pre-made exploits.

#### 💥 3. Exploitation (The Breach)
The attacker uses the information gathered to execute an attack against a known weakness.
* **Examples:** Exploiting an unpatched web service vulnerability, running an SQL Injection attack, or launching an ARP Spoofing attack on the local subnet.
* **SOC Visibility Indicator:** Anomalous protocol payloads, unauthorized remote connection attempts, or unusual traffic flows in Wireshark.

#### 🎯 4. Post-Exploitation (Control & Persistence)
After gaining an initial foothold, the attacker moves to secure their position.
* **Objective:** Establish persistent backdoor access, move laterally into other sensitive internal networks, and begin data exfiltration (stealing data).

---

### 🧩 Connecting the Architecture: The Security Matrix

Every tool and protocol learned so far serves a distinct purpose within this lifecycle:

| Concept / Tool | Role in the Attack Lifecycle | SOC Analytical Value |
| :--- | :--- | :--- |
| **DNS** | Locating and identifying target domains. | Pinpoints what external entities assets are contacting. |
| **Ports** | Identifying open entry points (doors) into a system. | Defines the available attack surface. |
| **TCP Handshake** | Establishing the base connection protocol. | Helps identify port scanning vs. legitimate connections. |
| **ARP Protocol** | Local network identity management (IP to MAC). | Used to spot local Man-in-the-Middle (MITM) hijacking. |
| **Nmap** | Active scanning and reconnaissance utility. | Simulates attacker probing behavior. |
| **Wireshark** | Full packet visibility and traffic monitoring. | Allows real-time analysis of the active attack stage. |

> 🧠 **The SOC Analyst Mindset:** When analyzing an active incident, you must always ask: *"Where are we currently positioned on the attack chain?"* Catching an attacker during **Reconnaissance** prevents a breach entirely. Catching them during **Post-Exploitation** turns into a high-priority incident response and containment operation.
>
> Add Day 4 Notes


## 📅 Day 5: Behavioral Traffic Analysis (Detecting Attacks in Real-Time)

### 📊 Core Concept: Signatures vs. Behavioral Anomalies
A packet analysis tool like **Wireshark** will almost never pop up with a flashing red light saying *"You are being hacked!"* Instead, security analysts must develop an eye for identifying **patterns that deviate from baseline normal behavior**. 

The transition from a network administrator to a SOC analyst requires a fundamental shift in perspective: **moving from packet reading to behavioral analysis.**

---

### 🚨 3 Major Detection Patterns Every Analyst Must Know

#### 🚪 1. Detecting Port Scanning (Reconnaissance)
* **The Pattern:** A single source IP addresses a massive volume of distinct destination ports within a microsecond window.
* **The Visual Indicator in Wireshark:** Endless lines of `SYN` flags with a complete absence of subsequent `SYN-ACK` or `ACK` responses.
* **Analyst Action Matrix:** 1. Isolate the scanning source IP.
  2. Map out exactly which ports are being targeted.
  3. Update firewall policies to block or heavily monitor that specific IP infrastructure.

#### 🧬 2. Detecting ARP Spoofing (Local MitM)
* **The Pattern:** Conflict in network device identity. The local network gateway IP address suddenly begins flapping between two completely different hardware addresses.
* **The Visual Indicator in Wireshark:** Multiple unsolicited ARP replies claiming ownership over the same internal gateway IP, paired with warning indicators of duplicate IP/MAC assignments.
* **The Threat:** An active Man-in-the-Middle (MitM) attack where an attacker is positioning themselves to sniff or modify data in transit.

#### 🌐 3. Detecting DNS Anomalies (Malware Communication)
* **The Pattern:** Spikes in highly unusual or structurally broken domain resolution queries.
* **The Visual Indicator in Wireshark:** Repeated queries for long, random alphanumeric strings (e.g., `x7k2j9-login-auth-verify.net`) or massive spikes in failed domain lookups.
* **The Threat:** This often indicates **Command and Control (C2) beaconing** or malware utilizing a Domain Generation Algorithm (DGA) to establish a back-channel connection to an external threat actor.

---

### 🧠 The Analyst Rulebook: Normal vs. Suspicious Traffic

To maintain network visibility, a SOC analyst checks real-time traffic against this standard baseline grid:

| Traffic Characteristic | Normal Baseline Behavior | Suspicious/Anomalous Behavior |
| :--- | :--- | :--- |
| **Packet Flow Volumetrics** | Steady, predictable streams of data. | Sudden bursts or floods of rapid, identical packets. |
| **Connection State** | Complete TCP handshakes (`SYN` ➔ `SYN-ACK` ➔ `ACK`). | Fragmented or half-open connections (`SYN` floods). |
| **Domain Navigation** | Valid, trusted domains mapping to static web services. | Random, obfuscated domain strings with high failure rates. |

> 📊 **SOC Operational Takeaway:** Attackers hide in plain sight by using your network's native protocols against you. They don't generate "corrupted" packets; they generate **abnormal behavioral volumes and sequences** using perfectly valid protocols.


> Add Day 5 Notes


## 📅 Day 6: The Incident Response Lifecycle (The Analyst Workflow)

### 🧠 The Mindset Shift: From Analysis to Investigation
A junior technician looks at network logs as isolated events. A professional **SOC Analyst** views logs through the lens of an investigation workflow. The ultimate rule of the security mindset is simple: 

> ❌ **Incorrect:** *"I saw a malicious packet."* >  **Correct:** *"I identified a pattern of anomalous behavior, mapped it to a specific stage of the attack lifecycle, and determined its operational meaning."*

---

### 🔁 The 5-Step Security Engineer Workflow

Every security investigation—whether in a small lab or an enterprise Security Operations Center (SOC)—follows this structured cycle:[Observe] ➔ [Identify] ➔ [Analyze] ➔ [Confirm] ➔ [Respond]
#### 👀 1. Observe (Data Collection)
* **Action:** Continuously gather and monitor live environmental data.
* **SOC Execution:** Using packet capture tools (Wireshark), firewalls, and logging systems to monitor raw packets, DNS queries, TCP connection states, and ARP broadcasts.

#### 🏷️ 2. Identify (Classification)
* **Action:** Classify and label the incoming traffic pattern.
* **SOC Execution:** Sorting traffic into high-level categories (e.g., *Is this standard web traffic, a potential scan, internal routing noise, or suspicious lookup traffic?*). You aren't fixing the issue yet—you are triage-labeling it.

#### 🔍 3. Analyze (Deep Inspection)
* **Action:** Drill deep into the technical metrics of the flagged traffic.
* **SOC Execution:** Inspecting specific **Source/Destination IPs**, targeted **Port numbers**, protocol handshakes, packet frequency, and timestamp distribution. You are answering: *"Does this traffic timing and structure make operational sense?"*

#### 🚨 4. Confirm (The Verdict)
* **Action:** Compare findings against known threat matrices to verify an actual attack.
* **SOC Execution:** Correlating the evidence to differentiate a false positive from a true threat:
  * **Normal Indicator:** Completed TCP handshakes to standard web ports (80/443).
  * **Attack Indicator:** Incomplete `SYN` floods targeting hundreds of sequential doors.

#### 🛡️ 5. Respond (Defensive Action)
* **Action:** Execute containment and mitigation strategies.
* **SOC Execution:** Isolating the affected internal asset, blacklisting the malicious external source IP at the firewall, raising an incident ticket, and preserving the packet logs for forensics.

---

### 🧩 System Integration: How the Foundation Connects

Your entire networking foundation serves an explicit purpose within this unified workflow:

| Concept / Tool | Operational Role in the Analyst Workflow |
| :--- | :--- |
| **Wireshark** | **Observe:** Provides complete, unfiltered network visibility. |
| **DNS Traffic** | **Identify:** Immediately maps out what external services an asset is calling. |
| **TCP Protocol** | **Analyze:** Allows the analyst to check connection health and completeness. |
| **Ports** | **Identify:** Signals exactly what services are being targeted or exposed. |
| **ARP Logs** | **Detect:** Pinpoints local identity-spoofing and subnet tampering. |
| **Nmap Mindset** | **Anticipate:** Gives the defender insight into how an attacker probes a network. |

---

### 🧪 Incident Walkthrough Blueprint (The Workflow in Action)

When presented with an alert, the workflow executes systematically:

1. **Observe:** Raw traffic spikes are flagged in the packet capture console.
2. **Identify:** High-density, repetitive outbound communication is classified as network probing.
3. **Analyze:** Verification shows one single external IP sending thousands of rapid `SYN` requests to sequential ports with zero `ACK` replies.
4. **Confirm:** Verified as an active **Reconnaissance Port Scan**.
5. **Respond:** Construct a firewall rule to drop all incoming traffic from the source IP and log the event.

Add Day 6 Notes - Workflow Complete   


## 📅 Day 7: Building a Sandboxed Cybersecurity Practice Lab

### 🧪 Core Concept: The Isolated Lab Environment
A professional security engineer never runs tests, active scans, or exploit payloads on live production networks or unauthorized internet systems. Instead, they leverage **Virtualization** to construct safe, legally compliant, and completely isolated sandbox environments.

---

### 🛠️ The Architecture of a Home Security Lab

To simulate network traffic, reconnaissance, and defensive monitoring safely, a standard virtualization architecture utilizes a hypervisor (such as **Oracle VirtualBox** or **VMware Workstation Player**) to run multiple independent operating systems on a single physical machine.



* **Virtual Machine (VM):** A software-defined computer running inside your physical hardware, possessing its own independent Operating System, virtual network interface card (vNIC), and localized IP address space.
* **The Snapshot Advantage:** VMs allow analysts to take "snapshots"—saving the exact state of the machine. If a system is intentionally infected with malware or corrupted during a testing scenario, it can be reverted to a clean state instantly.

---

### 🖥️ Lab Deployment Blueprint

A basic functional security lab requires two primary nodes operating on a controlled virtual network:
+-----------------------------------------------------------------+
|                       VIRTUAL HYPERVISOR                        |
|                                                                 |
|  [ Attacker/Analyst Node ]              [ Target Node ]         |
|       (Kali Linux)                       (Metasploitable/Win)   |
|       IP: 192.168.56.101                 IP: 192.168.56.102     |
|               |                                  |              |
+---------------+----------------------------------+--------------+
|                                  |
+--- [ Host-Only / Internal LAN ] -+1. **Node 1: The Attacker / Analyst Workstation (Kali Linux)**
   * **Purpose:** Acts as the primary operations console. Kali comes pre-packaged with essential offensive and defensive utilities, including `Wireshark`, `Nmap`, `tcpdump`, and packet manipulation tools.
2. **Node 2: The Target Machine (Ubuntu, Windows, or Metasploitable)**
   * **Purpose:** Acts as the asset being evaluated. This machine runs the services, open ports, or web applications that the analyst targets and monitors.

#### 🌐 Critical Network Configuration: Host-Only / Internal Network
> ⚠️ **Mandatory Security Constraint:** The virtual network interface for both VMs must be configured to **Host-Only** or **Internal Network**. This explicitly cuts off the VMs from accessing the physical local area network (LAN) and the public internet. This guarantees that all scanning, probing, and exploitation traffic remains entirely contained inside the virtual hypervisor.

---

### 🔬 Practical Hands-On Exercises

Once the lab environment is initialized, three fundamental validation exercises establish baseline visibility:

#### 1. Connectivity Verification (ICMP Ping)
* **Execution:** Run `ping [TARGET_IP]` from the Kali terminal.
* **Analyst Objective:** Verify that the two virtual instances can locate each other across the virtual switch using basic ICMP request and reply loops.

#### 2. Live Packet Capture Activation
* **Execution:** Launch `Wireshark` within Kali Linux and bind the capture engine to the active local interface (e.g., `eth0`).
* **Analyst Objective:** Keep Wireshark recording in the background while executing network actions to observe real-world, unencrypted protocol generation in real time.

#### 3. Targeted Reconnaissance Probing
* **Execution:** Run a localized scan using `nmap [TARGET_IP]`.
* **Analyst Objective:** Observe the rapid generation of half-open `SYN` connection frames inside your Wireshark window, verifying exactly how a port scan populates packet logs during an active reconnaissance phase.

---

### ⚖️ The Ethics of Security Engineering
> 📜 **The Golden Rule:** Only scan, probe, or test systems that **you explicitly own** or have received **direct, written authorization** to evaluate. Unauthorized network scanning against external internet infrastructure is illegal and easily flagged by enterprise intrusion detection systems. True professionals master their trade inside controlled environments.

Add Day 7 Notes - Lab Architecture Complete
