# Wazuh-Suricata Integration Lab

## Objective
Deployed and integrated Suricata Network Intrusion Detection System (NIDS) with Wazuh SIEM to create a comprehensive security monitoring solution. This project demonstrates the ability to implement network-based threat detection, centralized log management, and automated security alerting in a virtualized environment.

## Skills Demonstrated
- Network Intrusion Detection System (NIDS) deployment
- SIEM configuration and log aggregation
- Security event correlation and analysis
- Virtual network architecture design
- Linux system administration
- Configuration management
- Threat detection rule implementation

## Tools & Technologies Used
- **SIEM**: Wazuh (Open-source XDR platform)
- **NIDS**: Suricata 6.0.8
- **Virtualization**: Proxmox VE
- **Operating System**: Ubuntu 22.04 LTS
- **Threat Intelligence**: Emerging Threats ruleset
- **Log Format**: JSON (EVE format)

## Architecture

### Network Topology
```
                              ┌──────────────────────────┐
                              │   Physical Network       │
                              │   Router/Gateway         │
                              │   IP: 192.168.XXX.X      │
                              └────────────┬─────────────┘
                                           │
                     ┌─────────────────────┼─────────────────────┐
                     │                     │                     │
                     │ WiFi                │ Ethernet            │
                     │                     │                     │
          ┌──────────▼──────────┐   ┌──────▼──────────────────────────────┐
                Desktop PC                Proxmox Host (Physical Server)   
               (Windows 11)                       vmbr0 Bridge
                                                    PROMISC           
            Physical Endpoint       └──────┬───────────┬──────────┬───────┘
                                           │           │          │
            WiFi: 192.168.XXX.Y            │           │          │
                                    ┌──────▼─────┐ ┌───▼────┐ ┌──▼────────┐
              Wazuh Agent #3          Wazuh Mgr     Suricata    Parrot OS 
                 Monitored              (SIEM)       (IDS)      (Pwnbox)  
          └─────────────────────┘                                  
                                        enp6s18      enp6s19     enp6s20   
                                        .XXX.Y       .XXX.X      .XXX.Z    
                                                                
                                        Manager        IDS       Agent #2 
                                       Dashboard      Agent1     HTB Work 
                                        Indexer      PROMISC        
                                    └─────┬──────┘ └───┬─────┘ └────┬──────┘
                                          │            │            │
                                          │    Wazuh Agents         │
                                          │  Forward Logs/Alerts    │
                                          └────────────┴────────────┘
```

### Component Details
- **Wazuh Manager VM**: Central SIEM server collecting and analyzing security events
- **Suricata VM**: Dedicated network monitoring sensor in promiscuous mode
- **Monitored Endpoints**: Desktop and VM endpoints with Wazuh agents installed

## Implementation Steps

### 1. Suricata Deployment
- Installed Suricata 6.0.8 on Ubuntu 22.04 dedicated VM
- Configured network interface in promiscuous mode for network-wide visibility
- Deployed Emerging Threats ruleset for comprehensive threat detection
- Configured EVE JSON logging for structured event output

### 2. Wazuh Agent Configuration
- Installed Wazuh agent on Suricata VM
- Configured agent to forward Suricata EVE logs to Wazuh manager
- Established secure agent-manager communication via TCP/1514

### 3. SIEM Integration
- Configured Wazuh to parse Suricata JSON logs
- Enabled Suricata decoders and correlation rules
- Created custom dashboards for network threat visualization

### 4. Testing & Validation
- Generated test traffic (ICMP, HTTP, DNS)
- Verified alert generation in Wazuh dashboard
- Confirmed end-to-end log pipeline functionality

## Key Configuration Files

### Suricata Configuration (`/etc/suricata/suricata.yaml`)
```yaml
vars:
  address-groups:
    HOME_NET: "[10.10.10.0/24]"
    EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules
rule-files:
  - "*.rules"

af-packet:
  - interface: vmbr0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```

### Wazuh Agent Configuration (`/var/ossec/etc/ossec.conf`)
```xml
<ossec_config>
  <client>
    <server>
      <address>10.10.10.254</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
  </client>

  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
```

## Monitoring Capabilities

### Detection Coverage
- **Network Attacks**: Scans, exploits, malicious payloads
- **Protocol Anomalies**: Malformed packets, suspicious patterns
- **Command & Control**: Known C2 communication signatures
- **Malware Traffic**: Known malware network indicators
- **Policy Violations**: Unauthorized protocols, suspicious destinations

### Alert Categories
- ICMP traffic analysis
- HTTP/HTTPS inspection
- DNS query monitoring
- SSH brute force detection
- Port scanning identification

## Results & Outcomes
- Successfully deployed network-wide intrusion detection
- Achieved centralized visibility of network security events
- Implemented automated alerting for suspicious network activity
- Created foundation for incident detection and response capabilities
- Demonstrated ability to integrate multiple security tools

## Lessons Learned
- Importance of network interface promiscuous mode for comprehensive monitoring
- Value of structured logging (JSON) for SIEM integration
- Critical role of threat intelligence feeds (Emerging Threats)
- Need for proper network segmentation in virtual environments
- Significance of testing alert pipelines end-to-end

## Future Enhancements
- [ ] Implement custom Suricata rules for environment-specific threats
- [ ] Add additional network sensors for multi-segment monitoring
- [ ] Integrate with threat intelligence platforms (MISP, AlienVault OTX)
- [ ] Develop automated response playbooks for common threats
- [ ] Implement traffic baseline analysis and anomaly detection
- [ ] Add packet capture (PCAP) functionality for forensic analysis

## References
- [Wazuh Documentation - Network IDS Integration](https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html)
- [Suricata User Guide](https://docs.suricata.io/)
- [Emerging Threats Ruleset](https://rules.emergingthreats.net/)

## Environment Specifications
- **Hypervisor**: Proxmox VE
- **Suricata VM**: 2 vCPU, 2GB RAM, 20GB storage
- **Wazuh Manager**: OVA deployment
- **Network**: Single bridge (vmbr0) - 192.168.254.0/24

## Note
All IP addresses shown have been replaced with placeholders for my own peace of mind.
