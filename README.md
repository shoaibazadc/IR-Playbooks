# IR-Playbook-Library - Playbooks, Procedures & Compliance

This playbook library covers the full incident response lifecycle: detection criteria, triage logic, containment procedures, escalation paths and post-incident reporting, all built from simulated attacks in an isolated homelab SOC environment.

---

## Architecture

---

## Technologies

| Component | Role | Version |
|-----------|------|---------|
| **Wazuh** | SIEM / XDR - log ingestion, rule-based detection, alerting | 4.14 |
| **TheHive** | Case management - incident tracking and observables | 5.6 |
| **Sysmon** | Deep Windows endpoint telemetry (SwiftOnSecurity config) | 15.2 |
| **VirtualBox** | Hypervisor - hosted isolated internal network | 7.1 |
| **Kali Linux** | Attacker VM - Metasploit, msfvenom, netcat | 2026.1 |

---

## Playbook Coverage

| Scenario | Severity | MITRE Techniques | ICO Notifiable |
|----------|----------|-----------------|----------------|
| [Data Exfiltration](playbooks/data-exfiltration/playbook.md) | Critical | T1048, T1041, T1567 | Yes |
| [Ransomware](playbooks/ransomware/playbook.md) | Critical | T1486, T1490, T1489 | Yes |
| [Malware](playbooks/malware/playbook.md) | Critical | T1059, T1071, T1055, T1547 | Conditional |
| [Lateral Movement](playbooks/lateral-movement/playbook.md) | Critical | T1021.002, T1550.002, T1078 | Conditional |
| [Phishing](playbooks/phishing/playbook.md) | High | T1566.001, T1204.002, T1078 | Conditional |
| [Brute Force](playbooks/brute-force/playbook.md) | High | T1110.001, T1110.003, T1078 | Conditional |

---

## Project Phases

### Phase 1 - Infrastructure
Deployed Wazuh and TheHive in an Ubuntu Server VM (`soc-core`). Configured an isolated VirtualBox NAT network (`soc-net`, `10.0.5.0/24`) with port-forwarding rules to access the services on the host.

### Phase 2 - Endpoint Configuration
Deployed victim VMs as Wazuh agents alongside an attacker VM. Installed Sysmon with the SwiftOnSecurity config on the Windows machine and configured Wazuh to ingest `Microsoft-Windows-Sysmon/Operational` event channel logs.

### Phase 3 - Attack Simulation
Ran each scenario on the victim VMs before writing the corresponding playbook. Tools and techniques used per scenario:

```
Phishing       -> Malicious .hta / macro-enabled Word doc  -> Sysmon Event ID 1, 3
Malware        -> Metasploit meterpreter reverse shell     -> Sysmon Event ID 1, 3
Ransomware     -> RanSim / Python file encryptor           -> Wazuh FIM mass modification
Lateral Move   -> Metasploit PSExec / pass-the-hash        -> Event ID 4624, 7045
Brute Force    -> Hydra credential spray against RDP/SMB   -> Event ID 4625, 4740
Exfiltration   -> netcat large file transfer               -> Sysmon Event ID 3
```

### Phase 4 - Detection and Triage
Triaged the incident for each scenario: created cases, added observables and performed IOC enrichment. Also Documented Wazuh rule IDs, Sysmon events and Wazuh dashboard queries used during the investigation.

### Phase 5 - Playbook Creation
Wrote each playbook based on what happened in the lab. Each playbook contains:

- **Detection criteria** - Wazuh rule IDs and Sysmon event IDs that trigger the playbook
- **Triage questions** - first five questions to answer within five minutes of alert
- **Decision tree** - Mermaid diagram with branching containment logic
- **Escalation matrix** - L1 -> L2 -> L3 -> CISO triggers and handover requirements
- **ISO 27001 lifecycle phases** - Identification, Protection, Detection, Response and Recovery
- **Recovery validation checklist** - sign-off criteria before returning to production
- **UK GDPR / ICO compliance** - 72-hour notification obligations flagged per scenario
- **Known detection gaps** - what the stack misses and what would close the gap
- **Automation scripts** - shell and PowerShell for containment, evidence collection, threat hunting
- **IOC template** - pre-formatted for TheHive ingestion
- **Real-world reference** - UK-relevant incident tied to each scenario

Post-incident reports completed per scenario using the standard template.

---

## Repository Structure

```
ir-playbook-library/                      
├── docs/
│   ├── setup.md                        # Full deployment guide
│   ├── architecture.png                # Environment diagram
│   └── screenshots/
│       ├── sysmon-process-tree.png     
│       ├── thehive-case-phishing.png   
│       ├── thehive-case-observable.png 
│       ├── virustotal-hash-result.png  
│       ├── wazuh-dashboard-overview.png
│       ├── wazuh-fim-alert.png         
│       ├── wazuh-brute-force-alert.png 
│       └── wazuh-logon-type3.png  
├── playbooks/
│   ├── phishing/
│   │   ├── playbook.md                
│   │   └── collect-evidence.sh        # Evidence collection script
│   ├── brute-force/
│   │   ├── playbook.md                
│   │   └── hunt-failed-logons.ps1     # Failed logon hunting script
│   ├── ransomware/
│   │   ├── playbook.md                
│   │   ├── isolate-host.sh            # Host isolation script
│   │   └── hunt-encrypted-files.ps1   # Encrypted file detection script
│   ├── malware/
│   │   ├── playbook.md
│   │   ├── volatile-collection.ps1    # Volatile memory collection script
│   │   └── isolate-host.sh            # Host isolation script
│   ├─── data-exfiltration/
│   │   ├── playbook.md
│   │   └── hunt-staging-files.ps1     # Staged file and large transfer hunting script
│   └── lateral-movement/
│       ├── playbook.md                
│       └── hunt-logon-type3.ps1       # Remote logon hunting script
├── templates/
│   ├── playbook-template.md          
│   ├── post-incident-report-template.md
│   └── escalation-matrix-template.md
├── reports/
│   └── scenario-results.md            # Detection / response outcome
└── README.md     
```

---

## Screenshots

| Screenshot | Description |
|------------|-------------|
| `wazuh-dashboard-overview.png` | Wazuh dashboard showing live agent telemetry and alert activity |
| `wazuh-fim-alert.png` | FIM mass modification alert firing during ransomware simulation |
| `wazuh-brute-force-alert.png` | Wazuh alert on Event ID 4625 repeated failed logons during brute force simulation |
| `wazuh-logon-type3.png` | Wazuh alert on Event ID 4624 Logon Type 3 during lateral movement simulation |
| `thehive-case-phishing.png` | TheHive case with observables and enrichment results |
| `thehive-case-observable.png` | TheHive observable enrichment showing IOC lookup results |
| `virustotal-hash-result.png` | VirusTotal lookup on malware hash |
| `sysmon-process-tree.png` | Sysmon Event ID 1 process tree during malware simulation |

> All screenshots are in [`docs/screenshots/`](docs/screenshots/)

---

## Detection Gaps

This stack's known detection limitations are as follows:

- **Raw TCP exfiltration** is not detected by Wazuh alone - Zeek `conn.log` large transfer alerting would close this gap
- **Encrypted C2 over HTTPS** blends into normal web traffic - SSL inspection or Zeek JA3 fingerprinting required
- **Phishing email delivery** is not visible in Wazuh without mail gateway integration
- **No automated enrichment** - Cortex would automate the manual VirusTotal and AbuseIPDB lookups performed during each investigation

---

## Skills Demonstrated

- **Incident Response** - end-to-end case lifecycle across six MITRE ATT&CK-mapped scenarios
- **Threat Intelligence** - IOC enrichment via VirusTotal and AbuseIPDB
- **Adversary Emulation** - attack simulation using Metasploit, msfvenom and netcat
- **UK Regulatory Compliance** - ICO notification obligations and ISO 27001 lifecycle phases integrated into each playbook
- **Scripting & Automation** - PowerShell and Bash for host isolation, evidence collection and threat hunting

---

## Setup

Full deployment notes are in [`docs/setup.md`](docs/setup.md).

Requirements:
- Host machine with 16 GB+ RAM recommended
- VirtualBox 7.1
- Internet access

---

## References

- [ISO 27001:2022 - Information Security Management](https://www.iso.org/standard/27001)
- [NCSC Incident Management Guidance](https://www.ncsc.gov.uk/collection/incident-management)
- [ICO - Guide to UK GDPR](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [NHS WannaCry Post-Incident Report - NAO (2018)](https://www.nao.org.uk/reports/investigation-wannacry-cyber-attack-and-the-nhs/)
- [British Airways ICO Enforcement Notice (2020)](https://ico.org.uk/action-weve-taken/enforcement/british-airways/)
- [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
