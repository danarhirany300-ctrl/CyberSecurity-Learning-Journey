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
