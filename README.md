# Wazuh SOC Home Lab — SSH Brute Force Detection & Analysis

A self-built SOC home lab used to simulate, detect, and analyze an SSH brute-force attack using Wazuh (SIEM) and Suricata (network IDS), running fully on VirtualBox.

## Summary

This lab covers the full attacker → detection → analysis cycle:

- Built a two-VM environment (Kali Linux attacker, Ubuntu Server as the Wazuh manager/target)
- Deployed Wazuh (manager, indexer, dashboard) and integrated Suricata as a network IDS feeding into Wazuh
- Ran network reconnaissance (Nmap) against the target
- Ran a credential brute-force attack (Hydra) against the exposed SSH service
- Analyzed the resulting Wazuh alerts, mapped them to MITRE ATT&CK, and reasoned through how a SOC L1 analyst would triage them

## Architecture

| Host | Role | IP | Notes |
|---|---|---|---|
| Kali Linux | Attacker | 192.168.0.7 | Static IP, VirtualBox VM |
| Ubuntu Server 24.04 | Wazuh Manager + Target | 192.168.0.100 | Static IP, hosts Wazuh manager/indexer/dashboard + Suricata IDS |
| Windows 11 | Hypervisor host | 192.168.0.1 | Physical host running VirtualBox |

Full details: [`docs/01-lab-architecture.md`](docs/01-lab-architecture.md)

> **Note:** The Wazuh manager and the attack target are the same machine (192.168.0.100). Alerts show `agent.id: 000`, meaning the manager is self-monitoring its own `auth.log` and Suricata `eve.json` — there is no separate Windows/Ubuntu agent involved in this exercise.

## Attack → Detection Summary

| Phase | Tool | Action | Detected? | Rule ID | MITRE ATT&CK |
|---|---|---|---|---|---|
| Recon | Nmap | OS/service/port scan of the network, including the target | ✅ Yes (via Suricata) | 86601 | T1046 – Network Service Discovery |
| Exploitation | Hydra | SSH credential brute force against 192.168.0.100 | ✅ Yes (via Wazuh sshd/PAM rules) | 2502, 5760, 40112 | T1110 – Brute Force, T1078 – Valid Accounts |

Full write-ups:
1. [Lab Architecture](docs/1.lab-architecture.md)
2. [Wazuh Server Setup](docs/2.wazuh-server-setup.md)
3. [Network Reconnaissance](docs/3.network-reconnaissance.md)
4. [SSH Brute Force Simulation](docs/4.ssh-bruteforce-simulation.md)
5. [Detection & Analysis](docs/5.detection-analysis.md)
6. [Suricata IDS Integration](docs/6.suricata-ids-integration.md)
7. [Troubleshooting Log](docs/7.troubleshooting-log.md)

## Repo Structure

```
wazuh-soc-homelab/
├── README.md
├── docs/                     → all write-ups
├── configs/                  → Wazuh manager config + Suricata config used
└── evidence/
    ├── alerts/                → raw Wazuh alert JSON
    └── screenshots/           → terminal & dashboard screenshots
```

## Disclaimer

All activity in this lab was performed against systems I own, on an isolated home network, for learning purposes. Credentials shown in any evidence have been redacted.
