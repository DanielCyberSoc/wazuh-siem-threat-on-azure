# **Basic Cloud Threat Detection Setup: Wazuh SIEM on Azure**

## **Project Overview**
This project provides a detailed guide for integrating Wazuh, an open-source cybersecurity platform, with Microsoft Azure for effective threat detection and security monitoring. The integration aims to leverage Wazuh's capabilities in a cloud environment, enabling organizations to enhance their security posture and respond to incidents effectively.

---

## **Table of Contents**

1. [Prerequisites](https://github.com/DanielCyberSoc/wazuh-siem-threat-on-azure/edit/main/README.md#1-prerequisites)
2. [Virtual Machines Setup](https://github.com/DanielCyberSoc/wazuh-siem-threat-on-azure/edit/main/README.md#2-virtual-machines-setup)
    - [Linux VM (Wazuh Manager)](#linux-vm-wazuh-manager)
    - [Windows VM (Wazuh Agent)](#windows-vm-wazuh-agent)
3. [Wazuh Installation](#wazuh-installation)
    - [Installing Wazuh Manager](#installing-wazuh-manager)
    - [Installing Wazuh Agent](#installing-wazuh-agent)
4. [Deployment, Monitoring, and Alerting](#deployment-monitoring-and-alerting)
    - [Log Collection and Configuration](#log-collection-and-configuration)
    - [Monitoring Alerts](#monitoring-alerts)
    - [Validating Communication Between Manager and Agent](#validating-communication-between-manager-and-agent)
5. [Conclusion](#conclusion)

---

## **1. Prerequisites**

To complete this project, you’ll need:

- An **Azure subscription**.
- A **Linux Virtual Machine** running on Azure, which will host the Wazuh Manager.
- A **Windows Virtual Machine** running on Azure, which will host the Wazuh Agent.
- A **basic understanding of SIEM concepts** and Wazuh configurations.

---

## **2. Virtual Machines Setup**

### **2.1 Linux VM (Wazuh Manager)**

1. Create a Linux Virtual Machine in Azure (Ubuntu 20.04 or later is recommended).  
   - Assign a static public IP address for accessibility.  
   - Ensure SSH is enabled during setup.  
   - Recommended size: **Standard_B2s** for basic workloads.

![Linux VM Creation Screen](https://github.com/user-attachments/assets/f34c667c-8d27-4d61-b909-791f19187fb2)

---

### **2.2 Windows VM (Wazuh Agent)**

1. Create a Windows Virtual Machine in Azure (Windows Server 2019 or later is recommended).  
   - Configure RDP access and enable a static public IP address.  
   - Ensure the machine has at least 2 vCPUs and 4GB RAM for optimal performance.  

![Windows VM Creation Screen](https://github.com/user-attachments/assets/a6f3dfaa-e2fb-4177-9392-c5f4b1c16a6a)


2. Enable the following ports for communication between the VMs:
   - TCP port 1514 for syslog.
   - TCP port 1515 for the Wazuh agent connection.

---

3. Native SSH from your local machine
   ```bash
   ssh -i ~/.ssh/id_rsa.pem <your admin name>@<your public IP address>
   ```
## **3. Wazuh Installation**

### **3.1 Installing cURL for Ubuntu Linux**

1. Update the Linux VM:
   ```bash
   sudo apt update && sudo apt upgrade
   ```

2. Next, install cURL, execute:
   ```bash
   sudo apt install curl
   ```
---

### **3.2 Installing Wazuh**

1. Download and run the Wazuh installation assistant.
   ```bash
   curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
   ```
   Once the assistant finishes the installation, the output shows the access credentials and a message that confirms that the installation was successful.
   ```bash
   INFO: --- Summary ---
   INFO: You can access the web interface https://<WAZUH_DASHBOARD_IP_ADDRESS>
    User: admin
    Password: <ADMIN_PASSWORD>
   INFO: Installation finished.
   ```
2. Verify the installation:
   ```bash
   sudo systemctl status wazuh-manager
   ```
   ![Wazuh Manager Installation](https://github.com/user-attachments/assets/0c256234-cc91-4c8a-8d99-a577f756cadd)

### **3.3 Native RDP To Your Windows VM**

1. Open Remote Desktop Connection (RDP) on your local machine.

2. Enter the public IP address of the Windows VM and use the credentials configured during VM creation.
   
   ![Remote Desktop Connection](https://github.com/user-attachments/assets/d1f5545c-934c-435f-8232-915b228e07a6)
3. Access the Wazuh Dashboard via the correct Private IP address
   - Go to Linux VM and copy the Private IP address
   - Open web browser in Windows VM > https://<PrivateIPaddress>
   
   ![image](https://github.com/user-attachments/assets/5d042d4e-beac-4054-88b1-ae97f9e74a1e)

   - Click Advanced > Continue to <PrivateIPAddress>
   
   ![image](https://github.com/user-attachments/assets/db3b1d25-9c90-49c8-bf71-b8d2d542c597)

   -Login with your credential we got from Installing Wazuh
   
   ```bash
   INFO: --- Summary ---
   INFO: You can access the web interface https://<WAZUH_DASHBOARD_IP_ADDRESS>
    User: admin
    Password: <ADMIN_PASSWORD>
   INFO: Installation finished.
   ```


### **3.4 Installing Wazuh Agent**

1. Download the Wazuh Agent on the Windows VM from the [Wazuh downloads page](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html).

2. Configure the agent to communicate with the Wazuh Manager:
   - Input the server's IP address (e.g., 10.0.0.4) in the agent configuration > save the settings
   - Start the service. Once the agent is running, you can return to the dashboard to verify its status.

![Wazuh Agent Configuration](https://github.com/user-attachments/assets/b118cc04-35d5-45db-aa0e-244c42069129)
---
3. Verify Agent Connection:

Return to the Wazuh dashboard and navigate to Agents.
Check the agent's status to confirm it is actively reporting data

![image](https://github.com/user-attachments/assets/51a8b84f-b481-4bec-a616-b433f0def601)

## **4. Deployment, Monitoring, and Alerting**

### **4.1 Log Collection and Configuration**

1. Configure the Wazuh Manager to collect logs from both the Linux and Windows VMs.
   - Example of adding a local file to `/var/ossec/etc/ossec.conf`:
     ```xml
     <localfile>
        <log_format>syslog</log_format>
        <location>/var/log/auth.log</location>
     </localfile>
     ```

2. For Windows, ensure event logs are forwarded to the Wazuh Manager.

![Log Collection Configuration](path/to/log-collection-configuration-image.png)

---

### **4.2 Monitoring Alerts**

1. **Navigate to Threat Intelligence:**
   - Log in to the Wazuh dashboard (`https://<your-server-ip>`).
   - On the dashboard, go to **Threat Hunting** > **Events**.

2. **Filter Events:**
   - Use the filters to narrow down events by:
     - **Agent Name:** Select your agent (e.g., `wazuh-agent-vm`).
     - **Rule Description:** Look for logon failures or brute-force-related alerts.

3. **Analyze Brute-Force Attempts:**
   - Identify failed login attempts categorized under rule.id:
     - **Queries- 60122**
   - Review the timestamp, source IPs, and frequency of the attempts.

4. **Visualize Data:**
   - View the timeline chart for an overview of brute-force activity.
   - Identify patterns or specific timeframes with a spike in activity.

5. **Document Observations:**
   - Example: The screenshot with 689 available fields below shows failed login attempts detected by Wazuh in 24hours.

   ![Failed Login Attempts](https://github.com/user-attachments/assets/ab6aea6e-8738-45de-aba8-0a8290c0ae24)

---

## Conclusion

This project demonstrated the integration of Wazuh with Microsoft Azure to create a **basic cloud-based threat detection setup**. From deploying VMs to monitoring brute-force attempts, this guide provides a foundation for further exploration of SIEM capabilities.

