# Methodology

This assessment followed a structured ethical hacking workflow within an isolated lab environment.

## 1. Scope Definition

The assessment was limited to intentionally vulnerable systems inside a VirtualBox Host-Only network.

Out of scope:
- Public IP addresses
- Real organizations
- Third-party systems
- Production environments

## 2. Host Discovery

Nmap was used to identify active hosts within the lab subnet.

## 3. Service Enumeration

Version detection was performed to identify open ports, running services, and exposed software versions.

## 4. SMB Enumeration

Enum4linux was used to assess SMB exposure and identify information disclosure risks.

## 5. Vulnerability Analysis

Discovered services were reviewed for known vulnerabilities, insecure configurations, and excessive exposure.

## 6. Controlled Validation

Selected findings were validated only inside the authorized lab environment.

## 7. Reporting

Findings were documented with risk levels, evidence, impact, and remediation recommendations.
