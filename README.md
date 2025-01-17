# **Basic Cloud Threat Detection Setup: Wazuh SIEM on Azure**

## **Project Overview**
This project provides a detailed guide for integrating Wazuh, an open-source cybersecurity platform, with Microsoft Azure for effective threat detection and security monitoring. The integration aims to leverage Wazuh's capabilities in a cloud environment, enabling organizations to enhance their security posture and respond to incidents effectively.

---

## **Table of Contents**

1. [Prerequisites](#prerequisites)
2. [Virtual Machines Setup](#virtual-machines-setup)
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

![Linux VM Creation Screen](path/to/linux-vm-creation-image.png)

---

### **2.2 Windows VM (Wazuh Agent)**

1. Create a Windows Virtual Machine in Azure (Windows Server 2019 or later is recommended).  
   - Configure RDP access and enable a static public IP address.  
   - Ensure the machine has at least 2 vCPUs and 4GB RAM for optimal performance.  

![Windows VM Creation Screen](path/to/windows-vm-creation-image.png)

2. Enable the following ports for communication between the VMs:
   - TCP port 1514 for syslog.
   - TCP port 1515 for the Wazuh agent connection.

---

## **3. Wazuh Installation**

### **3.1 Installing Wazuh Manager**

1. Update the Linux VM:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. Install Wazuh Manager:
   ```bash
   sudo apt-get install wazuh-manager
   ```

![Wazuh Manager Installation](path/to/wazuh-manager-installation-image.png)

3. Verify the installation:
   ```bash
   sudo systemctl status wazuh-manager
   ```

---

### **3.2 Installing Wazuh Agent**

1. Download the Wazuh Agent on the Windows VM from the [Wazuh downloads page](https://wazuh.com/downloads/).

2. Configure the agent to communicate with the Wazuh Manager:
   - Edit the `ossec.conf` file and specify the Manager’s IP:
     ```xml
     <manager>
        <address>Your_Wazuh_Manager_IP</address>
     </manager>
     ```

3. Start the Wazuh Agent service.

![Wazuh Agent Configuration](path/to/wazuh-agent-configuration-image.png)

---

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

1. Use the `alerts.log` file on the Wazuh Manager to view detected security events.
   - Location: `/var/ossec/logs/alerts/alerts.log`.

![Alert Monitoring](path/to/alert-monitoring-image.png)

---

### **4.3 Validating Communication Between Manager and Agent**

1. Verify that the Wazuh Agent on the Windows VM is communicating with the Wazuh Manager.
   - Run the following command on the Linux VM:
     ```bash
     sudo /var/ossec/bin/agent_control -l
     ```

![Agent Communication Validation](path/to/agent-communication-validation-image.png)

2. Confirm that logs from both VMs are being ingested and processed.

---

## **5. Conclusion**

This project demonstrates the process of setting up a **basic cloud threat detection system** using Wazuh SIEM on Azure. It highlights how to collect logs, configure alerts, and monitor security events effectively. While we did not complete rule-writing in this lab, the groundwork laid here can be extended for more advanced configurations in the future.

![Wazuh Dashboard](path/to/wazuh-dashboard-summary-image.png)

---

### **Next Steps**

- Explore writing custom decoders and rules in Wazuh to detect specific threats.
- Integrate with additional Azure services such as Azure Monitor or Microsoft Sentinel for advanced threat detection capabilities.
