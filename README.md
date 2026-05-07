# Hands-On Ethical Hacking Final Project: Penetration Testing Report

> **Educational penetration testing assessment conducted in an isolated VirtualBox host-only lab.**

This repository presents a polished, portfolio-ready penetration testing report for a controlled ethical hacking lab environment. The assessment compares multiple virtual machines with different security postures, including an intentionally vulnerable Metasploitable2 target and a more restricted Axigen-Ubuntu appliance.

## Project Summary

| Item | Details |
| --- | --- |
| Course | INFO 3171(S10) System Security |
| Institution | Kwantlen Polytechnic University |
| Assessment Date | April 15, 2026 |
| Lab Network | `192.168.56.0/24` VirtualBox Host-Only |
| Assessment Type | Grey-box reconnaissance and controlled validation |
| Primary Tools | Nmap, Netcat, Wget, Enum4linux, Metasploit |

## What This Project Demonstrates

- Safe lab design using an isolated VirtualBox host-only network.
- Host discovery, port scanning, and service/version enumeration.
- Web banner analysis and vulnerable application discovery.
- SMB null-session enumeration and weak password policy identification.
- Risk classification, exploitation validation, remediation planning, and executive reporting.

## Key Findings

| System | Open Ports | Critical Findings | Overall Risk |
| --- | ---: | ---: | --- |
| Metasploitable2 | 23 | 3 | **Critical** |
| Axigen-Ubuntu | 2 | 0 | **Medium** |
| Kali Linux | 0 | 0 | **Low** |
| VirtualBox Host | 5 | 0 | **Low** |

The highest-risk findings were concentrated on Metasploitable2 and included an unauthenticated root shell on port `1524`, the vsftpd `2.3.4` backdoor associated with CVE-2011-2523, weak SMB/null-session exposure, and a dangerously permissive password policy.

## Repository Contents

- [`REPORT.md`](REPORT.md) - Full penetration testing assessment report.
- [`README.md`](README.md) - Portfolio overview and project summary.

## Ethical and Legal Notice

This project is for educational and portfolio demonstration purposes only. All testing described in the report was performed against intentionally vulnerable virtual machines in an isolated lab network owned and controlled by the tester. Do not scan, enumerate, exploit, or attempt to access systems without explicit written authorization.

## Recommended Use on GitHub

To make this project stand out on GitHub:

1. Add sanitized screenshots under an `assets/` directory.
2. Reference those screenshots from the report where the figure placeholders appear.
3. Avoid publishing real credentials, password hashes, private IPs from real networks, or identifying third-party information.
4. Keep the report focused on methodology, impact, remediation, and lessons learned.
