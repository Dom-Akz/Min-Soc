# pfSense: NGFW and Network Segmentation

## 1. What this does

- Splits the lab into 5 zones behind one firewall: **ATTACK, DMZ, SERVERS, DB, SOC** (plus WAN).
- Forces a real pivot: the attacker can only ever reach **DMZ**. SERVERS, DB and SOC are unreachable from ATTACK at the firewall level — the only way in is through a compromised DMZ host.
- Runs **Suricata inline** on pfSense itself (the NGFW part): it doesn't just allow/deny by port, it inspects and can drop traffic based on attack signatures.
- Gives DHCP, DNS and NTP to DMZ and ATTACK; SERVERS, DB and SOC use static IPs only.
- Ships every firewall and IDS log to the SOC's ELK stack, so blocked and dropped attack traffic becomes evidence even if a host is later compromised and its local logs are wiped.

## 2. Topology

```
                  Internet (NAT)
                        |
                   [ WAN em0 ]
                        |
                  +------------+
                  |  pfSense   |   <- Suricata inline here = NGFW
                  +------------+
                   |    |    |    |    |
              [ATTACK][DMZ][SERVERS][DB][SOC]
               10.40    10.5   10.20  10.50 10.30
                 |        |      |      |     |
               Kali   web-srv  app+AD  db-srv  Wazuh/ELK/Suricata-mgmt
```

Allowed path: ATTACK → DMZ (web ports only) → SERVERS (app port) → DB (DB port).
Blocked and logged: ATTACK → SERVERS/DB/SOC, and everything → SOC except the Wazuh agent ports.

## 3. Interfaces

| Zone    | pfSense if | Network       | pfSense IP | VirtualBox network     |
| ------- | ---------- | ------------- | ---------- | ---------------------- |
| WAN     | em0        | NAT           | DHCP       | NAT                    |
| SOC     | em1        | 10.10.30.0/24 | 10.10.30.1 | Host-only `vboxnet2`   |
| DMZ     | em2        | 10.10.5.0/24  | 10.10.5.1  | Internal `net-dmz`     |
| SERVERS | em3        | 10.10.20.0/24 | 10.10.20.1 | Internal `net-servers` |
| DB      | em4        | 10.10.50.0/24 | 10.10.50.1 | Internal `net-db`      |
| ATTACK  | em5        | 10.10.40.0/24 | 10.10.40.1 | Internal `net-attack`  |

em4 and em5 need adding via `VBoxManage` — VirtualBox's GUI only shows 4 adapters. Full values and commands: `configs/pfsense/interfaces-dhcp-dns.md`.

## 4. Firewall policy (summary)

| From            | To               | Allowed                                                           |
| --------------- | ---------------- | ----------------------------------------------------------------- |
| ATTACK          | DMZ              | 80, 443 only                                                      |
| DMZ             | SERVERS          | app port only                                                     |
| SERVERS         | DB               | DB port only                                                      |
| Any zone        | SOC              | Wazuh agent ports only (1514/1515), if agents are installed there |
| ATTACK          | SERVERS, DB, SOC | **Blocked, logged**                                               |
| Everything else |                  | **Blocked, logged**                                               |

Full aliases and rule tables: `configs/pfsense/aliases-and-rules.md`.

## 5. NGFW / IPS layer

pfSense's stateful firewall alone is not a next-gen firewall — it only matches ports and IPs. The Suricata package installed directly on pfSense, running **inline on the WAN and DMZ interfaces**, adds deep packet inspection and can actively drop malicious traffic by signature (SQLi attempts, known exploit payloads, C2 patterns) before it ever reaches the web app. That inline inspection is what makes this setup NGFW-equivalent.

Setup steps: `configs/pfsense/ngfw-suricata-ips.md`.

## 6. Logging (evidence)

- pfSense firewall logs → ELK (`10.10.30.20:5140`, syslog).
- Suricata (inline, on pfSense) alerts → ELK via Filebeat.
- This is continuous and one-way. An attacker who reaches SERVERS or DB and clears local logs cannot erase what was already shipped to the SOC.

Settings, ISO 27001 control mapping, and backup: `configs/pfsense/logging-and-backup.md`.

## 7. File index

| File                                     | Contents                                  |
| ---------------------------------------- | ----------------------------------------- |
| `configs/pfsense/README.md`              | This overview                             |
| `docs/installation/02-pfsense.md`        | Step-by-step install and config runbook   |
| `configs/pfsense/interfaces-dhcp-dns.md` | Interface, DHCP, DNS, NTP values          |
| `configs/pfsense/aliases-and-rules.md`   | Aliases and firewall rules per zone       |
| `configs/pfsense/ngfw-suricata-ips.md`   | Inline Suricata / NGFW setup              |
| `configs/pfsense/logging-and-backup.md`  | Remote logging, ISO 27001 mapping, backup |
| `docs/architecture/addressing-plan.md`   | Full IP plan (single source of truth)     |
| `docs/architecture/network-diagram.md`   | Mermaid diagram of this topology          |

## 8. Progress checklist

- [x] pfSense installed, WAN (em0) and SOC/LAN (em1) working
- [ ] SOC readdressed to 10.10.30.1/24 (was 10.10.10.1/24 — zone numbering changed)
- [ ] Host-only `vboxnet2` readdressed to 10.10.30.2/24 to match
- [ ] em2 (DMZ), em3 (SERVERS) added via Interfaces → Assignments
- [ ] em4 (DB), em5 (ATTACK) added via `VBoxManage`, then assigned in the GUI
- [ ] DHCP/DNS configured for DMZ and ATTACK
- [ ] Aliases and firewall rules created
- [ ] Suricata package installed and running inline on WAN + DMZ
- [ ] Remote logging to ELK enabled, tested
- [ ] Sanitized `config.xml` committed
- [ ] Snapshot `pfsense-ngfw-baseline` taken
