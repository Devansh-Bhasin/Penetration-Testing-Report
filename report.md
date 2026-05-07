# Hands-On Ethical Hacking Final Project - Penetration Testing Report

**Student:** Devansh Bhasin  
**Course:** INFO 3171(S10) System Security  
**Institution:** Kwantlen Polytechnic University  
**Date:** April 15, 2026  
**Network:** `192.168.56.0/24` VirtualBox Host-Only  
**Assessment Type:** Grey-box reconnaissance and controlled validation

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Testing Methodology](#2-scope-and-testing-methodology)
3. [Network Architecture and Environment](#3-network-architecture-and-environment)
4. [Host Discovery and Enumeration](#4-host-discovery-and-enumeration)
5. [Detailed Service Identification](#5-detailed-service-identification)
6. [Web Application Assessment](#6-web-application-assessment)
7. [SMB and User Account Enumeration](#7-smb-and-user-account-enumeration)
8. [Vulnerability Assessment and Critical Findings](#8-vulnerability-assessment-and-critical-findings)
9. [Exploitation Validation](#9-exploitation-validation)
10. [Axigen-Ubuntu Security Assessment](#10-axigen-ubuntu-security-assessment)
11. [Security Recommendations and Remediation](#11-security-recommendations-and-remediation)
12. [Attack Scenario Analysis](#12-attack-scenario-analysis)
13. [Risk Assessment Matrix](#13-risk-assessment-matrix)
14. [Conclusion and Key Learnings](#14-conclusion-and-key-learnings)
15. [Appendices](#15-appendices)

---

## 1. Executive Summary

This report documents a penetration testing assessment conducted inside an isolated VirtualBox laboratory network. The assessment focused on three virtual machines and used a grey-box methodology combining reconnaissance, enumeration, vulnerability analysis, and controlled validation. All testing remained confined to the `192.168.56.0/24` host-only network to prevent interaction with external or production systems.

The assessed environment showed a clear contrast between systems with weak and strong security postures. Metasploitable2, the intentionally vulnerable target, exposed a large attack surface and multiple critical weaknesses. Axigen-Ubuntu demonstrated a significantly more restricted and production-oriented configuration. Kali Linux, used as the tester workstation, exposed minimal network services.

The most serious issues were concentrated on Metasploitable2:

- 23 open TCP ports.
- Multiple outdated services with known security weaknesses.
- SMB null-session enumeration exposing 34 user accounts.
- Vulnerable web applications exposed through the HTTP service.
- A weak password policy requiring only 5 characters and no complexity.
- An unauthenticated root shell on port `1524`.
- The vsftpd `2.3.4` backdoor associated with CVE-2011-2523.

If equivalent issues existed in a production environment, they would enable rapid full-system compromise. Metasploitable2 was therefore classified as **Critical** risk, Axigen-Ubuntu as **Medium** risk because of its sensitive email-server role, Kali Linux as **Low** risk, and the VirtualBox host interface as **Low** risk.

---

## 2. Scope and Testing Methodology

### 2.1 Assessment Objectives

The assessment objectives were to:

- Identify active systems on the isolated subnet.
- Enumerate open ports, active services, and software versions.
- Assess exposed web services and hosted vulnerable applications.
- Enumerate SMB configuration details, user accounts, shares, and password policy information.
- Classify risk and produce remediation recommendations.
- Validate selected critical findings in a controlled lab setting.

### 2.2 Testing Scope

#### In-Scope Targets

| Target | IP Address | Role |
| --- | --- | --- |
| Metasploitable2 | `192.168.56.101` | Intentionally vulnerable Linux target |
| Axigen-Ubuntu | `192.168.56.102` | Production-style email appliance |
| Kali Linux | `192.168.56.103` | Penetration testing workstation |
| VirtualBox Host Interface | `192.168.56.1` | Host-only network gateway/interface |

#### Out-of-Scope Items

- Any systems outside `192.168.56.0/24`.
- The public internet or external networks.
- Denial-of-service testing.
- Data destruction or unauthorized modification.
- Any system not owned or explicitly authorized for testing.

### 2.3 Methodology

The assessment followed a grey-box methodology with partial knowledge of the lab network. Testing progressed through:

1. Lab isolation and network validation.
2. Host discovery.
3. TCP service enumeration.
4. Web banner and content review.
5. SMB enumeration.
6. Vulnerability classification.
7. Controlled exploitation validation for selected critical findings.
8. Remediation planning and final reporting.

### 2.4 Tools Used

| Tool | Purpose |
| --- | --- |
| Nmap | Host discovery, port scanning, and service/version detection |
| Netcat | Manual TCP interaction and banner grabbing |
| Wget | Non-interactive web content retrieval |
| Enum4linux | SMB, user, share, and password policy enumeration |
| Metasploit | Controlled validation of known lab vulnerabilities |

---

## 3. Network Architecture and Environment

### 3.1 Why VirtualBox Host-Only Networking Was Used

VirtualBox host-only networking was selected because it creates an isolated internal network segment. Unlike bridged networking, host-only mode prevents the vulnerable lab machines from receiving addresses on the physical network and prevents accidental exposure to external systems.

This was especially important because Metasploitable2 is intentionally vulnerable and Kali Linux can generate aggressive scanning traffic. Host-only networking allowed the testing workstation and lab targets to communicate while keeping the assessment contained inside the hypervisor.

### 3.2 Network Design and Addressing

The lab used the `192.168.56.0/24` subnet. The VirtualBox host-only interface operated at `192.168.56.1`, and the virtual machines were assigned addresses in the same subnet.

| Address | System |
| --- | --- |
| `192.168.56.1` | VirtualBox host-only interface |
| `192.168.56.101` | Metasploitable2 |
| `192.168.56.102` | Axigen-Ubuntu |
| `192.168.56.103` | Kali Linux |

**Figure 1: ICMP Ping Sweep Confirming Host Availability**  
This test confirmed that multiple hosts were reachable within the subnet and that the lab systems could communicate over the isolated network.

### 3.3 System Roles

- **Kali Linux (`192.168.56.103`)** acted as the assessment workstation.
- **Metasploitable2 (`192.168.56.101`)** represented a deliberately vulnerable legacy system.
- **Axigen-Ubuntu (`192.168.56.102`)** represented a more restricted production-style appliance.
- **VirtualBox host interface (`192.168.56.1`)** provided the host-only network interface.

---

## 4. Host Discovery and Enumeration

### 4.1 Scanning Methodology

Initial discovery used Nmap service/version detection:

```bash
nmap -sV 192.168.56.0/24
```

The `-sV` option was selected because identifying a service by port number alone is insufficient. Version detection helps determine whether a service maps to known vulnerable software.

### 4.2 System Discovery Analysis

**Figure 2: Nmap Subnet Scan Identifying Active Hosts**  
The scan identified active hosts and established the initial attack surface for further analysis.

Metasploitable2 returned 23 open TCP ports, making it the primary target for deeper enumeration. Axigen-Ubuntu returned only 2 open ports, suggesting a much smaller exposed attack surface. Kali Linux showed no unnecessary exposed services, which aligned with a hardened assessment workstation baseline.

### 4.3 Network Topology Implications

The scan results indicated that traffic between lab systems was not blocked by internal firewalls or segmentation controls. This allowed full host-to-host testing within the isolated lab environment.

---

## 5. Detailed Service Identification

### 5.1 Critical Services on Metasploitable2

**Figure 3: Detailed Nmap Scan of Metasploitable2**  
The scan revealed numerous open ports and outdated services, substantially increasing the system's attack surface.

Important services included:

| Port | Service | Security Concern |
| ---: | --- | --- |
| 21 | vsftpd 2.3.4 | Known backdoored version associated with CVE-2011-2523 |
| 22 | OpenSSH 4.7p1 | Legacy SSH implementation |
| 23 | Telnet | Cleartext remote login protocol |
| 53 | ISC BIND 9.4.2 | Legacy DNS service exposure |
| 80 | Apache 2.2.8 | Unsupported web server version |
| 139/445 | Samba 3.x | SMB enumeration and legacy configuration risk |
| 3306 | MySQL | Database exposed to the network |
| 5432 | PostgreSQL | Database exposed to the network |
| 6000 | X11 | Legacy graphical service with high exposure risk |
| 6667 | UnrealIRCd | Unusual service for this system role |

### 5.2 Infrastructure Services

The exposure of DNS, SMB, and database services created multiple paths for enumeration and compromise. In a production architecture, databases should normally be restricted to trusted application servers or internal network segments rather than exposed broadly to a flat subnet.

### 5.3 Specialized and Unusual Services

Java RMI, X11, and IRC services significantly expanded the attack surface. These services were not required for the target's core purpose and demonstrated the risk of feature bloat.

### 5.4 Service Analysis Summary

Metasploitable2 demonstrated a deliberately poor security posture. The system exposed many services, several of which were outdated or intentionally vulnerable. An attacker would not require a zero-day exploit because known vulnerabilities and weak configurations were already present.

---

## 6. Web Application Assessment

### 6.1 HTTP Server Banner Extraction

Netcat was used to manually request HTTP headers from port `80`:

```bash
nc -nv 192.168.56.101 80
HEAD / HTTP/1.0
```

**Figure 5: Netcat Banner Grabbing of Web Server**  
The server response disclosed Apache and PHP version information, including Apache `2.2.8` and PHP `5.2.4` package details.

This type of banner disclosure assists attackers by revealing exact software versions that can be mapped to known vulnerabilities.

### 6.2 Vulnerable Application Discovery

Wget was used to retrieve web content from the server root:

```bash
wget http://192.168.56.101/
```

**Figure 6: Wget Retrieval of Web Content**  
The retrieved content exposed links to intentionally vulnerable applications, including common training platforms such as DVWA, Mutillidae, and phpMyAdmin.

In a production environment, intentionally vulnerable applications should never be deployed or exposed. Even in a lab, they should remain isolated and access-controlled.

---

## 7. SMB and User Account Enumeration

### 7.1 Samba Service and Null Sessions

Ports `139` and `445` were open on Metasploitable2, so SMB enumeration was performed with Enum4linux. The system allowed unauthenticated SMB interaction through a null session.

A null session allows a client to interact with SMB without valid credentials. This is a serious information disclosure risk because it can expose users, groups, shares, and policies.

### 7.2 User Account Analysis

**Figure 7: SMB Enumeration via Enum4linux**  
Enum4linux retrieved system information, Samba version details, and 34 valid user accounts through unauthenticated enumeration.

Exposed account names included high-value accounts such as `root`, `postgres`, `mysql`, and `msfadmin`. Valid usernames reduce the complexity of password attacks because an attacker no longer needs to guess account names.

### 7.3 Share Analysis and the `/tmp` Exposure

**Figure 8: SMB Share Enumeration and Weak Password Policy**  
The SMB results exposed multiple shares, including a `tmp` share mapped to the system `/tmp` directory.

Anonymous read/write access to `/tmp` can allow attackers to stage malicious files. Even if file upload alone does not provide code execution, it can become part of an attack chain when combined with another vulnerability.

### 7.4 Weak Password Policy

**Figure 9: Weak Password Policy Identified via SMB Enumeration**  
The enumerated password policy showed:

- Minimum password length of 5 characters.
- No complexity requirement.
- No meaningful account lockout threshold.

Combined with the exposed list of valid usernames, this policy would make credential attacks highly likely to succeed.

---

## 8. Vulnerability Assessment and Critical Findings

### 8.1 Vulnerability Classification

Findings were categorized based on impact, likelihood, and ease of exploitation. Critical issues were those that could independently lead to unauthenticated system compromise or make credential compromise trivial.

### 8.2 Critical Vulnerabilities

| Finding | Evidence | Impact |
| --- | --- | --- |
| vsftpd 2.3.4 backdoor | Service identified on port `21` | Unauthenticated remote command execution |
| Bind shell on port `1524` | Direct Netcat connection returned root shell | Immediate root-level access |
| Weak password policy | SMB enumeration showed 5-character minimum and no complexity | High likelihood of credential compromise |

### 8.3 High-Risk Vulnerabilities

High-risk issues included:

- Apache `2.2.8` exposure.
- Samba `3.0.20` exposure.
- Legacy OpenSSH.
- Exposed databases.
- Intentionally vulnerable web applications.
- Unnecessary services such as Telnet, X11, Java RMI, and IRC.

### 8.4 Vulnerability Interdependencies

The greatest risk came from how the findings could be chained. SMB enumeration exposed usernames, weak password policy enabled password attacks, writable shares could stage payloads, and vulnerable services provided multiple execution paths. This is a classic defense-in-depth failure.

---

## 9. Exploitation Validation

> Validation was performed only inside the isolated lab environment against intentionally vulnerable systems.

### 9.1 Unauthenticated Root Shell - Port `1524`

Netcat was used to connect directly to port `1524` on Metasploitable2:

```bash
nc -nv 192.168.56.101 1524
whoami
id
```

**Figure 10: Unauthenticated Root Shell via Port 1524**  
The connection returned an immediate shell without authentication. `whoami` returned `root`, and `id` confirmed UID `0`.

**Impact:** Complete system takeover in seconds without credentials.

### 9.2 FTP Backdoor - CVE-2011-2523

Metasploit was used to validate the vsftpd `2.3.4` backdoor in the lab:

```text
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
run
```

**Figure 11: NVD Entry for CVE-2011-2523**  
The vulnerability is associated with the compromised vsftpd `2.3.4` release.

**Figure 12: Exploitation of vsftpd Backdoor**  
The exploit opened a command shell running with root privileges.

**Impact:** Malicious code embedded in a trusted software package allowed unauthenticated root compromise.

### 9.3 Exploitation Summary

Both validated paths provided immediate root-level access. The port `1524` bind shell and the vsftpd backdoor required no credential guessing, no password cracking, and no advanced exploitation technique.

---

## 10. Axigen-Ubuntu Security Assessment

### 10.1 Positive Security Characteristics

Axigen-Ubuntu exposed only two ports to the lab subnet, including SSH and a management interface. This demonstrated strong service minimization compared with Metasploitable2.

The SSH service used a modern OpenSSH version, and standard mail transport ports were not broadly exposed in the scan results. This reduced the externally visible attack surface.

### 10.2 Risk Assessment

Axigen-Ubuntu was classified as **Medium** risk. Although no critical findings were identified, the system represents a high-value target because email servers can contain sensitive communications and administrative interfaces.

### 10.3 Comparison with Metasploitable2

The comparison shows the value of service minimization and patch currency. Axigen-Ubuntu limited its externally visible services, while Metasploitable2 exposed many unnecessary and outdated services.

---

## 11. Security Recommendations and Remediation

### 11.1 Immediate Critical Actions

1. **Remove vsftpd 2.3.4**
   - Stop and disable the service.
   - Replace it with a maintained FTP/SFTP solution if file transfer is required.
   - Prefer SFTP over FTP to avoid cleartext credential transmission.

2. **Remove the port `1524` bind shell**
   - Identify the parent process binding to the port.
   - Remove the related `inetd` or `xinetd` configuration.
   - Block the port at the host firewall.

3. **Strengthen password policy**
   - Enforce at least 12 characters.
   - Require complexity or passphrase standards.
   - Enable account lockout or throttling.
   - Audit existing credentials.

### 11.2 Medium-Term Improvements

- Upgrade Apache to a supported `2.4.x` release or newer.
- Upgrade OpenSSH to a current supported version.
- Upgrade Samba to a supported `4.x` release.
- Disable SMBv1 and enforce secure SMB settings.
- Restrict database ports to trusted hosts only.
- Disable Telnet, IRC, X11, Java RMI, and other unnecessary services.
- Deploy host firewall rules with default-deny inbound policy.

### 11.3 Long-Term Strategic Improvements

- Implement regular patch management.
- Segment servers by function and sensitivity.
- Remove training applications from any production-like network.
- Deploy logging, monitoring, and alerting.
- Perform recurring vulnerability scans and annual penetration tests.
- Maintain asset inventory and configuration baselines.

---

## 12. Attack Scenario Analysis

### Scenario 1 - Unauthenticated Shell Access via Port `1524`

An attacker identifies port `1524` during reconnaissance and connects directly with Netcat. The service returns a root shell without authentication. The attacker gains full filesystem access and can use the compromised host for lateral movement.

### Scenario 2 - FTP Backdoor Exploitation

An attacker identifies vsftpd `2.3.4` on port `21`, maps it to CVE-2011-2523, and uses a public exploit module. The result is unauthenticated root-level command execution.

### Scenario 3 - Web Application Exploitation Chain

An attacker browses to the web server and discovers vulnerable applications such as DVWA or Mutillidae. Application vulnerabilities could be chained with weak local configuration, file upload, credential disclosure, or local privilege escalation to achieve broader compromise.

---

## 13. Risk Assessment Matrix

| System | Open Ports | Critical Findings | High Findings | Medium Findings | Risk Level | Mitigation Priority |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| Metasploitable2 | 23 | 3 | 8 | 6 | **Critical** | Rebuild or isolate |
| Axigen-Ubuntu | 2 | 0 | 0 | 0 | **Medium** | Monitor actively |
| Kali Linux | 0 | 0 | 0 | 0 | **Low** | Maintain baseline |
| VirtualBox Host | 5 | 0 | 0 | 2 | **Low** | Review access |

---

## 14. Conclusion and Key Learnings

This assessment demonstrated how basic reconnaissance and enumeration can reveal enough information to compromise a poorly configured system. Metasploitable2 exposed multiple independent paths to full compromise, including an unauthenticated root shell, a known FTP backdoor, and weak authentication controls.

Key lessons include:

- **Defense in depth matters:** A single failure should not result in full compromise.
- **Patch currency is critical:** Outdated services create predictable exploitation paths.
- **Information disclosure helps attackers:** Service banners and SMB enumeration reduce attacker effort.
- **Service minimization reduces risk:** Axigen-Ubuntu demonstrated a much smaller exposed attack surface.
- **Authentication policy matters:** Weak passwords and no lockout transform exposed usernames into a practical compromise path.

Organizations should prioritize patching, least privilege, service reduction, strong authentication, segmentation, and continuous monitoring.

---

## 15. Appendices

### Appendix A - Tool Documentation

| Tool | Description |
| --- | --- |
| Nmap | Network scanning and service/version detection |
| Netcat | Raw TCP/UDP socket interaction and banner grabbing |
| Wget | Non-interactive web content retrieval |
| Enum4linux | SMB enumeration utility |
| Metasploit | Exploit validation framework for authorized testing |

### Appendix B - Vulnerability References

- National Vulnerability Database (NVD)
- Common Vulnerability Scoring System (CVSS)
- Exploit Database
- Vendor security advisories

### Appendix C - Ethics and Legal Considerations

- Obtain explicit written authorization before testing.
- Test only approved systems and IP ranges.
- Avoid denial-of-service or destructive actions unless explicitly authorized.
- Protect sensitive data discovered during testing.
- Report findings responsibly and securely.
