<div align="center">

# 🛡️ Mini SOC — Detection & Incident Response Architecture

**A segmented, virtualized Security Operations Center lab: host & network intrusion detection, centralized log analytics, MITRE ATT&CK-aligned detection engineering, and attack-driven validation.**

![Status](https://img.shields.io/badge/status-in%20progress-orange)
![SIEM/XDR](https://img.shields.io/badge/XDR-Wazuh-blue)
![NIDS](https://img.shields.io/badge/NIDS-Suricata-red)
![Analytics](https://img.shields.io/badge/analytics-ELK-yellow)
![Firewall](https://img.shields.io/badge/firewall-pfSense-lightgrey)
![Framework](https://img.shields.io/badge/framework-MITRE%20ATT%26CK-black)

_Author: Soufiane Moussaoui — Cybersecurity Engineering Student (EMSI) — [github.com/Dom-Akz](https://github.com/Dom-Akz)_

</div>

---

## 📑 Table of Contents

1. [Project Overview](#1-project-overview)

---

## 1. Project Overview

**Mini SOC** is a home-lab Security Operations Center built entirely in a virtualized environment. It reproduces, at small scale, the core capabilities of an enterprise SOC:

| SOC Capability                             | Implementation in this lab                                          |
| ------------------------------------------ | ------------------------------------------------------------------- |
| Network segmentation & perimeter control   | **pfSense** (VLANs, inter-VLAN firewall policy)                     |
| Host-based detection (HIDS/EDR-like)       | **Wazuh** agents on Windows (with **Sysmon**) and Linux             |
| Network-based detection (NIDS)             | **Suricata** sensor                                                 |
| Log centralization, search & visualization | **ELK Stack** (Elasticsearch, Logstash, Kibana)                     |
| Detection engineering                      | Custom rules aligned with **MITRE ATT&CK**                          |
| Adversary emulation / validation           | **Kali Linux** + **Atomic Red Team**                                |
| Incident response                          | Documented playbooks following the NIST incident handling lifecycle |

> **Project status:** 🚧 _In progress._ This document is the **design baseline** (architecture & conception). Installation and configuration details are tracked in [Section 14](#14-installation--configuration-guide) and will be completed as each component is deployed.

### 1.1 Why this project?

Security teams do not just _collect_ logs — they must **detect**, **triage**, **investigate**, and **respond**. This lab exists to practice that full chain end-to-end:

```
Attack  ─►  Telemetry  ─►  Detection  ─►  Alert  ─►  Triage  ─►  Response  ─►  Lessons learned
(Kali /     (Sysmon,       (Wazuh /       (Dashboards)  (Analyst)   (Playbooks /  (Rule tuning)
 Atomic)     Suricata,      Suricata                                 Active
             pfSense)       rules)                                   Response)
```
