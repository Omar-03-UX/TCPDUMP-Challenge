# TCPDUMP-Challenge
<h2>Description</h2>
Detecting Lumma Stealer Malware Activity

Overview
An alert was generated from an endpoint exhibiting **abnormal network behavior**, suggesting potential **info-stealer malware activity**.

To investigate the incident, I analyzed the contents of the **PCAP file** to determine the nature of the communication and identify any potential **Indicators of Compromise (IOCs)**.

The analysis revealed activity consistent with the **Lumma Stealer malware**, where the infected endpoint communicated with external infrastructure and attempted to transfer data using **HTTP and FTP protocols**. Additionally, evidence suggested the attacker leveraged **Telegram infrastructure** for communication and file transfer.


# Investigation Findings

## 1. Packet Analysis

The investigation began by determining the total number of packets in the capture to understand the scope of the traffic.

After confirming packet volume, I examined the **most frequently appearing IP addresses** within the traffic. This helped identify the primary endpoint involved in the communication.

From this analysis:

- **10.0.2.10** was identified as the internal endpoint generating suspicious traffic.
- **171.161.116.100** was identified as a **malicious IP address** during OSINT analysis.
- **93.184.215.14** was flagged on VirusTotal as **phishing infrastructure**, raising further suspicion.
- **172.67.72.15** was identified as a **Cloudflare IP**, suggesting potential use of CDN infrastructure.

This step helped establish a baseline of **network communication patterns** and identify key external systems interacting with the endpoint.

## 2. HTTP Traffic Investigation

Next, HTTP traffic was examined to determine whether the endpoint was communicating with external servers using **web requests**.

During this analysis, only **one POST request** was observed directed toward **93.184.215.14**. This IP is associated with **Edgecast caching infrastructure**, which is commonly used for content delivery.

However, because malicious actors often leverage legitimate infrastructure for malicious activity, this connection remained suspicious.

## 3. Credential Discovery in HTTP Traffic

The payload of HTTP packets was then inspected for **possible credential leakage or authentication attempts**.

By searching packet payloads, I looked for keywords commonly associated with authentication fields, such as:

- user
- pass
- login

This technique is frequently used by SOC analysts to identify **credentials transmitted in clear text**.

## 4. Discovery of FTP Communication

Beyond HTTP traffic, additional inspection of packet payloads revealed **FTP communication occurring within the capture**.

FTP activity is particularly important because the protocol transmits data in **plain text**, meaning credentials and transferred data can often be recovered directly from packet captures.

This discovery suggested the attacker may have used FTP as a **file-sharing mechanism**.

---

## 5. Identification of Suspicious User-Agent

Further inspection revealed a suspicious user-agent string:

**“Tesla Browser”**

This raised suspicion because the user-agent does not correspond to standard browser patterns and is commonly associated with **malware activity or automated tools**.

OSINT investigation of the associated infrastructure revealed links to **Telegram Messenger infrastructure**, suggesting the attacker used Telegram as a **command-and-control or data exfiltration channel**.


## 6. Credential Extraction

After identifying FTP traffic, the packet capture was analyzed to determine **whether valid credentials were used to access the file-sharing server**.

By inspecting FTP commands within the packet payload, the login sequence could be observed, revealing the **credentials used by the endpoint**.

Because FTP transmits credentials in plain text, it was possible to recover them directly from the network traffic.

# Indicators of Compromise (IOCs)

| Indicator | Description |
| --- | --- |
| **10.0.2.10** | Compromised internal endpoint |
| **171.161.116.100** | Malicious external IP identified via OSINT |
| **93.184.215.14** | Suspicious HTTP destination flagged as phishing |
| **172.67.72.15** | Cloudflare infrastructure used in communication |
| **149.154.167.99** | Telegram infrastructure used by attacker |
| **User-Agent: Tesla Browser** | Suspicious user agent linked to malware activity |

# Key Commands Used During Investigation

### Count packets in the capture

Used to understand the scale of the traffic in the PCAP.

```
tcpdump -r tcpdump_challenge.pcap --count
```

---

### Identify most frequent IP addresses

Used to determine which systems were communicating most frequently.

```
tcpdump -tt -r tcpdump_challenge.pcap -n tcp | cut -d " " -f 3 | cut -d "." -f 1-4 | sort | uniq -c | sort -nr
```

---

### Identify HTTP POST requests

Used to identify potential data submissions or credential transmissions.

```
sudo tcpdump -r tcpdump_challenge.pcap -n tcp and port 80 | grep -E "POST"
```

---

### Search HTTP payloads for credentials

Used to detect potential username or password fields transmitted in clear text.

```
sudo tcpdump -tt -r tcpdump_challenge.pcap -n -A 'tcp port 80' | grep -iE "user|pass|login" | grep -v "User-Agent"
```

---

### Inspect all packet payloads

Used to manually examine traffic for suspicious activity.

```
sudo tcpdump -tt -r tcpdump_challenge.pcap -n -A
```

---

### Inspect traffic from the infected endpoint

Used to isolate activity originating from the compromised host.

```
tcpdump -tt -r tcpdump_challenge.pcap host 10.0.2.10 -A
```

---

### Extract FTP credentials

Used to identify authentication attempts within FTP traffic.

```
sudo tcpdump -tt -r tcpdump_challenge.pcap -n -A 'tcp port 21' | grep -iE "user|pass|login" | grep -v "User-Agent"
```

---

### Investigate Telegram infrastructure communication

Used to inspect traffic associated with Telegram infrastructure used by the attacker.

```
sudo tcpdump -nn -A -r tcpdump_challenge.pcap host 149.154.167.99
```

<br />
<h2> Utilities Used</h2>
- <b>VM WORKSTATION</b> 

<h2>Program walk-through:</h2>

<p align="center">
 <br/>
<br /> <img src="https://i.imgur.com/lJPRXaK.png" height="80%" width="80%" alt="Protocol"/>
<br />
<img src="https://i.imgur.com/Ubjwyhk.png" height="80%" width="80%" alt="Protocol"/>
<br />
<br />
<img src="https://i.imgur.com/2B4nkiE.png" height="80%" width="80%" alt="Source"/>
<br /> 
<img src="https://i.imgur.com/7BgyOzb.png" height="80%" width="80%" alt="Source"/>
<br />
