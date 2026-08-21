# Wazuh-SIEM-Multi-Endpoint-Security-Monitoring-Lab
A simulated enterprise security environment deploying an Ubuntu 24.04 Wazuh central manager to monitor Windows Server 2025 and Kali Linux endpoints. Configured agent-based logging, system activity tracking, and real-time threat detection across multi-OS assets.
                ┌──────────────────────────────┐
                │        Wazuh Manager          │
                │   Ubuntu 24.04 (VMware)       │
                │  - Wazuh Indexer               │
                │  - Wazuh Server                │
                │  - Wazuh Dashboard (port 443)  │
                │      192.168.131.131           │
                └───────────────┬───────────────┘
                                │
              ┌─────────────────┴─────────────────┐
              │                                     │
   ┌──────────▼──────────┐             ┌────────────▼───────────┐
   │   Kali Linux Agent   │             │  Windows Server Agent  │
   │   "LinuxUser"         │             │   "WindowsHost"         │
   │   192.168.131.128     │             │   192.168.131.132       │
   └────────────────────────┘             └──────────────────────────┘
