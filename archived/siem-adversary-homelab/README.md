# Adversary Emulation and SIEM Homelab

## Objective

This Wazuh Lab aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system, to mimic real-world attack scenarios. This hands-on experience was designed to deepen understanding of network security, attack patterns, and defensive strategies.

### Tools Used

- Wazuh as an SIEM.
- Ubuntu 22.04 Server for Wazuh.
- Windows 10 as Wazuh Agent Client.
- Ubuntu 22.04 Server as Wazuh Agent Client.
- Sysmon.
- Kali 2024.2 as Red team Operator.
- MITRE Caldera
- Wireshark.
## Construction

### Step 1:
Specifications:
</br>CPU: 4 cores
</br>RAM: 8GB+ 
</br>HDD: 50+ GB
</br>OS: Ubuntu 22.04 LTS

Download Ubuntu Server and after installation put in this command to install Wazuh Manager:

</br>curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh && sudo bash ./wazuh-install.sh -a with root rights.
</br>Write down name, password and website after installation.

### Step 2:
Specifications:
</br>CPU: 2 cores
</br>RAM: 4GB
</br>HDD: 50+ GB
</br>OS: Windows 10 Pro

Install Win10 and Download Sysmon.
Download Wazuh Agent and install it with this command:

</br>To deploy the Wazuh agent on your endpoint, edit the WAZUH_MANAGER variable to contain your Wazuh manager IP address or hostname.
</br>.\wazuh-agent-4.8.1-1.msi /q WAZUH_MANAGER="10.0.0.2"
</br>Login on Website and check connection.
</br>Install Wireshark and other preferred Tools like Vscode or SysinternalSuite on Win10 Client.

### Step 3:
Specifications:
</br>CPU: 2 cores
</br>RAM: 4GB
</br>HDD: 50+ GB
</br>OS: Ubuntu 22.04 Server

Install the GPG key:
</br>curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg

</br>Add the repository:
</br>echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list

</br>Update the package information:
</br>apt-get update

</br>Install Wazuh Agent:
</br>To deploy the Wazuh agent on your endpoint, edit the WAZUH_MANAGER variable to contain your Wazuh manager IP address or hostname.
</br>WAZUH_MANAGER="10.0.0.2" apt-get install wazuh-agent

</br>Enable services:
</br>systemctl daemon-reload
</br>systemctl enable wazuh-agent
</br>systemctl start wazuh-agent
</br>Login on Website and check connection.

### Step 4:
Specifications:
</br>CPU: 4 cores
</br>RAM: 8GB
</br>HDD: 50+ GB
</br>OS: Kali Linux 2024.2 

Create a Kali Linux VM and install MITRE Caldera with these commands:

</br>git clone https://github.com/mitre/caldera.git --recursive
</br>cd caldera
</br>pip3 install -r requirements.txt
</br>python3 server.py --insecure --build

</br>Add Windows 10 and Ubuntu Server 22.04 as Agents over MITRE Caldera.
</br>Commands to do that are in the “Agent” section from MITRE Caldera Dashboard. 
</br>Check TTP’s and test attack pattern on the two target systems as you see fit.

### Full Construction:
<div>
  <img src="/siem-adversary-homelab/siem_adversary_emulation_homelab/construction.png" alt="Full Construction">
</div>
