# Wazuh-SIEM-Multi-Endpoint-Security-Monitoring-Lab
A simulated enterprise security environment deploying an Ubuntu 24.04 Wazuh central manager to monitor Windows Server 2025 and Kali Linux endpoints. Configured agent-based logging, system activity tracking, and real-time threat detection across multi-OS assets.

| Component  | Specification / Parameter     
| ------- | ----------------------- | 
| Objective     |Stand up a central SIEM manager and onboard endpoints for detection and auditing |            
| Hypervisor      | VMware Workstation       
| Manager OS | Ubuntu 24.04 LTS (Wazuh Indexer, Server, and Dashboard single-node deployment)      | 
| Endpoints  | Kali Linux 2026.3 (LinuxUser), Windows Server 2025 Standard (WindowsHost)  |
| Wazuh Version  | 4.14.7 |


      ┌───────────────────────────────┐
                     │         Wazuh Manager         │
                     │      Ubuntu 24.04 (VMware)    │
                     │  - Wazuh Indexer              │
                     │  - Wazuh Server               │
                     │  - Wazuh Dashboard (Port 443) │
                     │        <manager-ip>           │
                     └───────────────┬───────────────┘
                                     │
                       ┌─────────────┴─────────────┐
                       │                           │
             ┌─────────▼──────────┐      ┌─────────▼───────────┐
             │   Kali Linux Agent │      │ Windows Server Agent│
             │     "LinuxUser"    │      │    "WindowsHost"    │
             │   <kali-agent-ip>  │      │ <windows-agent-ip>  │
             └────────────────────┘      └─────────────────────┘     


# Step 1: Deploy the Wazuh Manager (Ubuntu 24.04)
Deploy a fresh Ubuntu 24.04 VM inside VMware Workstation. The manager, indexer, and web dashboard can be installed together using Wazuh's automated unattended installer.

Run the scripts/install-wazuh-manager.sh script or execute:

Bash
`curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh
sudo bash wazuh-install.sh -a`

## Troubleshooting: Recovering from an Interrupted Package Installation
If an installation attempt fails or aborts halfway through, the wazuh-manager package can throw post-removal errors during reinstallation (`find: '/var/ossec/api/': No such file or directory`), breaking `dpkg`.

Instead of re-creating the Ubuntu VM from scratch, run scripts/reset-failed-manager-install.sh or execute:

Bash
`echo "exit 0" | sudo tee /var/lib/dpkg/info/wazuh-manager.postrm
sudo apt purge wazuh-manager -y
sudo dpkg --purge --force-remove-reinstreq wazuh-manager
sudo rm -rf /var/ossec`

## Re-run the installation script
`sudo bash wazuh-install.sh -a`
Upon a successful installation, the terminal prints the dashboard URL and the generated admin user credentials.

Security Note: Save the auto-generated password immediately to a password manager and avoid committing credentials to public version control repositories.

Log in to `https://<manager-ip>` to verify access to the dashboard interface.

# Step 2: Deploy the Kali Linux Agent
Onboard the Kali Linux endpoint to begin ingesting Linux system logs and process activity into the central manager.

Run `scripts/deploy-linux-agent.sh` or download the Debian package manually, passing parameters as environment variables:

Bash
`sudo bash -c "wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.7-1_amd64.deb \
  && WAZUH_MANAGER='<manager-ip>' WAZUH_AGENT_NAME='LinuxUser' dpkg -i ./wazuh-agent_4.14.7-1_amd64.deb"`
Enable and start the wazuh-agent service:

Bash
`sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent`
Alternatively, the same configuration string can be generated via Dashboard -> Deploy new agent wizard.


#   Step 3: Deploy the Windows Server Agent
Onboard the Windows Server 2025 instance to compare logging formats and security events across operating systems.

Open an elevated PowerShell session on the Windows Server target.

Run scripts/deploy-windows-agent.ps1 or download the .msi package directly:

PowerShell
`Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi 
  -OutFile $env:tmp\wazuh-agent.msi; 
msiexec.exe /i $env:tmp\wazuh-agent.msi /q WAZUH_MANAGER='<manager-ip>' WAZUH_AGENT_NAME='WindowsHost'`
Start the agent service:

PowerShell
`NET START Wazuh`

Step 4: Generate Telemetry (Recon via Kali)
To test the end-to-end detection flow, simulate basic external reconnaissance from the Kali Linux host against an internal network target (`<target-ip>`).

Run `scripts/recon-scan.sh` or execute:

Bash
## Service version scanning
`nmap -sV <target-ip>`

## OS identification scanning
`nmap -O <target-ip>`

# Step 5: Verify Agent Status & Dashboard Ingestion
Navigate to Wazuh Dashboard → Endpoints Summary to verify connectivity across both registered systems:

LinuxUser (`<kali-agent-ip>`) — Active

WindowsHost (`<windows-agent-ip>`) — Active

Review the Overview dashboard to watch real-time alert aggregation across built-in monitoring modules, including Threat Intelligence, PCI DSS, NIST 800-53, and System Auditing.

# Step 6: Security Configuration Assessment (SCA) 
* The results below are specific to my lab environment; your scores and failed checks will differ.

Under Wazuh Dashboard → Management → Configuration Assessment, review the baseline security metrics generated for `WindowsHost`.

Benchmark Applied: CIS Microsoft Windows Server 2025 Benchmark

Audit Results: 105 Passed | 293 Failed

Compliance Score: 26%
