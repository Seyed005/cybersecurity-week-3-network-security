# Week 3 --- Network Security Assessment

**Advanced Cybersecurity Practical \| Nmap • Wireshark • Vulnerability
Assessment • Security Hardening**

![Cybersecurity](https://img.shields.io/badge/Focus-Network%20Security-1f6feb)
![Lab](https://img.shields.io/badge/Environment-Authorized%20Isolated%20Lab-2ea043)
![Week](https://img.shields.io/badge/Internship-Week%203-8250df)

## 1. Project Overview

This repository contains the practical work completed for **Week 3 ---
Advanced Cybersecurity Practical Task**.

The assessment follows an evidence-driven workflow:

> **Discover → Enumerate → Capture → Investigate → Assess Risk → Harden
> → Verify**

All testing was performed against the intentionally vulnerable
**Metasploitable 2** virtual machine inside an isolated VirtualBox
host-only network.

**Important:** The scans and security testing documented here were
performed only against the authorized lab target.

------------------------------------------------------------------------

## 2. Student Details

  Field               Details
  ------------------- -------------------------
  Student Name        **Seyed Ismai Bilal S**
  Student ID          **016**
  Lab Workstation     Kali Linux
  Target              Metasploitable 2
  Target IP           `192.168.56.101`
  Kali Lab IP         `192.168.56.1`
  Lab Network         `192.168.56.0/24`
  Capture Interface   `vboxnet0`

------------------------------------------------------------------------

## 3. Objectives

The practical assessment focused on:

-   Network discovery and host identification.
-   Basic and advanced Nmap scanning.
-   Service and version enumeration.
-   Operating-system fingerprinting.
-   TCP SYN and UDP assessment.
-   NSE vulnerability identification.
-   Wireshark packet capture and protocol analysis.
-   Correlation of Nmap activity with network packets.
-   Vulnerability and risk prioritization.
-   Security hardening of unnecessary legacy services.
-   Before/after verification using repeat Nmap scans.
-   Maintaining professional evidence for the final assessment.

------------------------------------------------------------------------

## 4. Lab Architecture

``` text
                    Isolated VirtualBox Host-Only Network
                         192.168.56.0/24
                                |
                    +-----------+-----------+
                    |                       |
             Kali Linux               Metasploitable 2
             192.168.56.1             192.168.56.101
                    |                       |
              Nmap / Wireshark       Vulnerable Services
                    |
                vboxnet0
```

The environment was intentionally isolated so that reconnaissance,
traffic capture and hardening could be performed safely without
targeting public or third-party systems.

------------------------------------------------------------------------

## 5. Tools Used

### Nmap

Used for:

-   Host discovery
-   Port enumeration
-   Service/version detection
-   OS detection
-   TCP SYN scanning
-   UDP scanning
-   NSE vulnerability assessment
-   Default NSE enumeration
-   Before/after hardening verification

### Wireshark

Used for:

-   Packet capture
-   Protocol identification
-   Endpoint analysis
-   TCP handshake investigation
-   HTTP traffic inspection
-   DNS request/response analysis
-   ARP observation
-   TCP flag filtering

### VirtualBox

Used to maintain the isolated host-only lab network and run the
Metasploitable 2 target.

------------------------------------------------------------------------

## 6. Network Discovery --- Task 7

Host discovery was performed against the authorized lab network:

``` bash
sudo nmap -sn 192.168.56.0/24
```

The assessment identified three active hosts:

-   `192.168.56.1`
-   `192.168.56.100`
-   `192.168.56.101`

The primary assessment target was:

``` text
192.168.56.101 — Metasploitable 2
```

### Initial TCP exposure

A basic scan identified **23 open TCP ports** on the target.

Important exposed services included:

           Port Service
  ------------- ---------------------------
             21 FTP / vsftpd 2.3.4
             22 SSH
             23 Telnet
             25 SMTP
             53 DNS
             80 HTTP
        139/445 Samba
    512/513/514 R-services
           1099 Java RMI
           1524 Metasploitable root shell
           2049 NFS
           2121 ProFTPD
           3306 MySQL
           5432 PostgreSQL
           5900 VNC
           6667 UnrealIRCd
           8009 AJP13
           8180 Tomcat

------------------------------------------------------------------------

## 7. Advanced Nmap Assessment --- Task 8

The following assessment techniques were performed:

``` bash
nmap -sV 192.168.56.101
sudo nmap -O 192.168.56.101
sudo nmap -sS 192.168.56.101
sudo nmap -sU --top-ports 20 192.168.56.101
nmap --script vuln 192.168.56.101
nmap -sC 192.168.56.101
```

### Key enumeration results

Service/version detection identified several legacy services, including:

-   `vsftpd 2.3.4`
-   `OpenSSH 4.7p1`
-   `Apache httpd 2.2.8`
-   `BIND 9.4.2`
-   `Samba 3.X–4.X`
-   `MySQL 5.0.51a`
-   `PostgreSQL 8.3.x`
-   `VNC protocol 3.3`
-   `UnrealIRCd`
-   Apache Tomcat

OS detection indicated a **Linux 2.6.x** target.

------------------------------------------------------------------------

## 8. Vulnerability Assessment --- Task 12

The NSE vulnerability assessment reported multiple security weaknesses.

### High-priority observations

-   **vsftpd 2.3.4 backdoor --- CVE-2011-2523**
-   **OpenSSL CCS Injection --- CVE-2014-0224**
-   **SSL POODLE --- CVE-2014-3566**
-   **Logjam / weak Diffie-Hellman --- CVE-2015-4000**
-   Weak Diffie-Hellman configuration
-   Anonymous Diffie-Hellman MITM exposure
-   Java RMI classloader / remote-code-execution risk
-   UnrealIRCd backdoor/trojaned version indication

### Web and configuration observations

The scan also identified:

-   `/dvwa/`
-   `/mutillidae/`
-   `/phpMyAdmin/`
-   `/test/`
-   `/webdav/`
-   `/phpinfo.php`
-   TRACE method enabled
-   Session cookies without the HttpOnly flag in some cases
-   Slowloris vulnerability indication

These findings demonstrate the risk created by running old software and
unnecessary services on an exposed host.

------------------------------------------------------------------------

## 9. Wireshark Investigation --- Tasks 9--11

Wireshark was used on:

``` text
vboxnet0
```

### Protocols observed

-   ARP
-   ICMP
-   TCP
-   HTTP
-   DNS
-   BROWSER / broadcast traffic

The normal traffic capture contained **38 packets with 0 dropped**.

A separate Nmap capture contained approximately **2,027 packets with 0
dropped**.

### Useful display filters

``` text
dns
tcp
http
arp
tcp.flags.syn == 1
tcp.flags.reset == 1
```

### HTTP investigation

The HTTP exchange showed the expected sequence:

``` text
SYN → SYN/ACK → HTTP GET → HTTP 200 OK → connection termination
```

### DNS investigation

A DNS query for:

``` text
metasploitable.local
```

was observed from Kali to the target DNS service.

The response indicated a server failure.

### TCP reset observation

The dedicated Nmap capture contained reset traffic associated with scan
behavior.

The normal traffic capture did **not** contain packets matching:

``` text
tcp.flags.reset == 1
```

Therefore, the report records the distinction between the two captures
instead of treating them as the same evidence.

------------------------------------------------------------------------

## 10. Hardening --- Task 13

Security hardening focused on unnecessary legacy remote-access services.

### 10.1 FTP

Before hardening:

``` text
21/tcp open ftp vsftpd 2.3.4
```

The FTP service was disabled through the target's xinetd configuration.

After hardening:

``` text
21/tcp closed ftp
```

Evidence files:

``` text
04-Hardening/Before/FTP-Before.txt
04-Hardening/After/FTP-After.txt
```

### 10.2 Telnet

Telnet was disabled by commenting its service entry in:

``` text
/etc/inetd.conf
```

Before:

``` text
23/tcp open telnet
```

After:

``` text
23/tcp closed telnet
```

Evidence files:

``` text
04-Hardening/Before/Telnet-Before.txt
04-Hardening/After/Telnet-After.txt
```

### 10.3 R-services

The following legacy services were disabled:

``` text
512/tcp exec
513/tcp login
514/tcp shell
```

After hardening:

``` text
512/tcp closed exec
513/tcp closed login
514/tcp closed shell
```

Evidence files:

``` text
04-Hardening/Before/Rservices-Before.txt
04-Hardening/After/Rservices-After.txt
```

Configuration backups were created before editing the relevant
configuration files.

------------------------------------------------------------------------

## 11. Before / After Verification

The final verification scan was:

``` bash
sudo nmap -sS -sV 192.168.56.101
```

### Measured result

  Measurement        Before   After
  ---------------- -------- -------
  Open TCP ports         23      17
  Ports reduced         ---       6
  Reduction             ---   \~26%

The six removed exposures were:

-   FTP --- `21/tcp`
-   Telnet --- `23/tcp`
-   exec --- `512/tcp`
-   login --- `513/tcp`
-   shell --- `514/tcp`

The final scan still showed legacy services such as SMB, databases, VNC,
Tomcat, RMI and ProFTPD on port 2121. These remain candidates for
additional remediation in a production environment.

------------------------------------------------------------------------

## 12. Evidence Structure

The repository follows the required submission organization:

``` text
Week-3-Cybersecurity/
├── 01-Nmap/
│   ├── Scans/
│   └── Screenshots/
├── 02-Wireshark/
│   ├── PCAP/
│   └── Screenshots/
├── 03-Vulnerability-Assessment/
├── 04-Hardening/
│   ├── Before/
│   └── After/
├── Network-Diagram/
├── Week-3-Report.pdf
├── Week-3-Presentation.pptx
└── README.md
```

### Key evidence files

``` text
02-Wireshark/PCAP/Task10-Nmap-Wireshark.pcapng

03-Vulnerability-Assessment/
└── Task14-Final-Scan.txt

04-Hardening/Before/
├── FTP-Before.txt
├── Telnet-Before.txt
└── Rservices-Before.txt

04-Hardening/After/
├── FTP-After.txt
├── Telnet-After.txt
└── Rservices-After.txt
```

Screenshots and additional scan outputs are stored in their respective
folders.

------------------------------------------------------------------------

## 13. Professional Security Recommendations

1.  Remove unnecessary legacy services from production systems.
2.  Replace unsupported or vulnerable software with maintained versions.
3.  Restrict database, RMI, VNC and administrative services to trusted
    management networks.
4.  Disable insecure plaintext protocols such as Telnet and legacy
    remote-shell services.
5.  Use modern TLS configurations and eliminate weak cryptographic
    algorithms.
6.  Apply secure HTTP configuration and security headers.
7.  Review exposed web applications and administrative paths.
8.  Perform recurring vulnerability assessments after remediation.
9.  Retain packet captures and scan output as evidence for investigation
    and verification.
10. Maintain configuration backups before security changes.

------------------------------------------------------------------------

## 14. Security Lessons Learned

This practical demonstrated that network security assessment is not
limited to finding open ports.

A professional assessment connects:

-   **Port exposure** → what is reachable
-   **Service enumeration** → what is running
-   **Vulnerability assessment** → why it is risky
-   **Packet analysis** → what the traffic actually looks like
-   **Hardening** → how exposure can be reduced
-   **Verification** → whether the change actually worked

The measurable reduction from **23 to 17 open TCP ports** provided
concrete evidence that hardening reduced the target's attack surface.

------------------------------------------------------------------------

## 15. Conclusion

The Week 3 assessment successfully combined network discovery, advanced
Nmap enumeration, Wireshark traffic investigation, vulnerability
assessment and practical security hardening.

The assessment identified multiple legacy and vulnerable services on
Metasploitable 2. Three hardening areas were implemented and verified:

-   FTP
-   Telnet
-   R-services

The final scan confirmed that the selected services were no longer
exposed, reducing the number of open TCP ports from **23 to 17**.

This repository contains the supporting scan outputs, packet capture,
screenshots, network diagram, final report and presentation required for
the practical submission.

------------------------------------------------------------------------

## 16. Responsible Use

All reconnaissance, scanning, packet capture and hardening activities
documented in this repository were performed in an **authorized,
isolated cybersecurity lab**.

Do not use these procedures against systems, networks or services
without explicit authorization.
