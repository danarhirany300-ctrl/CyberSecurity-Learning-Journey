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
