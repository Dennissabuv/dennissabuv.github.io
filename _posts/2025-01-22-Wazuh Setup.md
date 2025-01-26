---
title: Installing Wazuh on RHEL.
categories: [Wazuh]
tags: [security,rhel,homeserver]
---

![HeadingDisplay]({{site.url }}/assets/posts/wazuh/HeadingDisplay.png)

### RedHat
In this post, I will provide a step-by-step guide to installing Wazuh on a Red Hat system. This process is simple and straightforward, ensuring that you can quickly set up Wazuh for enhanced security monitoring on your environment

The reason I choose Redhat is due to its widely recognized and stable Linux distribution used in enterprise environments. The certifications like RHCSA and RHCE are valuable and will be a good option to learn.With its strong community, extensive resources, and open-source contributions, Red Hat is an excellent choice for those looking to advance their careers in system administration, cloud computing, and DevOps.

### Wazuh

Wazuh is a security platform that provides unified XDR and SIEM protection for endpoints and cloud workloads. The solution is composed of a single universal agent and three central components: the Wazuh server, the Wazuh indexer, and the Wazuh dashboard

You may Read more - [Wazuh.com](https://documentation.wazuh.com/current/installation-guide/index.html)


### Here's the plan:
* Installing RedHat Enterprise on Proxmox
* Installing Wazuh Server on RHEL
* Deploying the agent


## 1. Installing Red Hat on Proxmox

- I have downloaded RHEL from Red Hat developer for free with the inidividial developer license

  You can read more and download - [RedHatDevelopers](https://developers.redhat.com/products/rhel/download)

- Upload the ISO and install RHEL on your Hypervisor / Server.
- Install using the recommended settings.

Once the server is ready, perform a the System package and upgrade.
Use the dnf utitily to update and upgrade the system.

  - #### Update the package repository cache

    This command ensures that the system has latest repository data.    
    ```bash
    sudo dnf makecache
    ```

  - #### Update all installed packages
    This command checks for updates to all installed packages and applies the latest versions if available.
    ```bash
    sudo dnf update -y
    # -y will automatically confirm without prompting for input
    ```

 -  #### Upgrade the system
    This command will upgrade all installed packages to their latest versions

    ```bash
    sudo dnf upgrade -y
    ```

 -  #### Clean the cache
    After updating or upgrading, you can clean up the local repository cache to free up disk space, this is optional.

    ```bash
    sudo dnf clean all
    ```

## 2. Installing WAZUH 

Detailed installation can be found by visiting their official documentation site: [Documentation](https://documentation.wazuh.com/current/quickstart.html)

I will be using the installation assistant to install the Wazuh Server.You may use the below script if you plan to install Wazuh central components on the same host. (Please visit the official Wazuh site for requirements)

 - 1 Download and run the Installation assistant
 ````bash
 curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i
 ````
Here, I have used - i to ignore the system requirements.

![WazuhInstall]({{site.url }}/assets/posts/wazuh/WazuhInstall01.png)

Please wait till the installation completes and look for any warnings, once the installation is completed, it will show the username and password to login to the portal.

        You can access the web interface https://<wazuh-dashboard-ip>:443

Here, the installation log showed a warning 

 WARNING: The system has Firewalld enabled. Please ensure that traffic is allowed on these ports: 1515, 1514, 443 and when I tried to access the dashboard, I get the below error.

![WazuhError]({{site.url }}/assets/posts/wazuh/WazuhError.png)

This leads us to the path that the firewall rules needs to be changed.


#### Firewall Changes

I checked the firewall status by running the below command and it shows that the ports are not allowed

```bash
sudo firewall-cmd --list-all
```

![WazuhFirewallStatus]({{site.url }}/assets/posts/wazuh/WazuhFirewallStatus.png)

 - We would need to allow the below ports

    * Port 1514 (TCP/UDP):This is the default port used by the Wazuh agent to send logs to the Wazuh manager. 

    * Port 1515 (TCP):This port is often used for the Wazuh cluster and also for enrollment service

    * Port 443 (TCP):Port 443 is the standard port for HTTPS traffic. Wazuh uses this port for secure communication, particularly for the Wazuh Dashboard (which uses Kibana and the Wazuh plugin).


Confirm that the firewall services are running 

```bash
sudo systemctl status firewalld
```
You will see listed as active, if the firewall services are not running you can start the service by running

```bash
sudo systemctl start firewalld
```
Allow the required ports

Use the firewall-cmd command to allow traffic on the specified ports. 
Since 1515, 1514, and 443 are the required ports, run the following commands.

Currently my default firewall zone is public and I will adding the changes to the public zone.

To display the current default zone

```bash
sudo firewall-cmd --get-default-zone
```
Adding the firewall rules



```bash
sudo firewall-cmd --zone=public --add-port=1515/tcp --permanent 
sudo firewall-cmd --zone=public --add-port=1514/tcp --permanent
sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
```

once the changes have been made,

Reload the firewall to apply the changes:

```bash
sudo firewall-cmd --reload
```

Verify the open ports

To verify that the ports have been opened successfully, you can list the active rules:

```bash
sudo firewall-cmd --list-all
```

Reload the web page and you will be able to login to the Wazuh dashboard using the credentials given at the time of installation
___

#### Changing Admin Password


If you want to change admin password , run the script with the -u|--user <USER> option and indicate the new password with the option -p|--password <PASSWORD>. If no password is specified, the script will generate a random one.
Wazuh Site Documentation : [PasswordChange_Wazuh](https://documentation.wazuh.com/current/user-manual/user-administration/password-management.html#passwords-distributed)

The passwords tool is embedded in the Wazuh indexer under :

<b>/usr/share/wazuh-indexer/plugins/opensearch-security/tools/</b>

You can use the embedded version or download it with the following command :

```bash
curl -so wazuh-passwords-tool.sh https://packages.wazuh.com/4.10/wazuh-passwords-tool.sh
#Run below to change the password
bash wazuh-passwords-tool.sh -u admin -p Secr3tP4ssw*rd
```
Output should be as below

```
                    #OUTPUT
25/01/2025 22:35:00 INFO: Updating the internal users.
25/01/2025 22:35:08 INFO: A backup of the internal users has been saved in the /etc/wazuh-indexer/internalusers-backup folder.
25/01/2025 22:35:08 INFO: Generating password hash
25/01/2025 22:35:10 INFO: The filebeat.yml file has been updated to use the Filebeat Keystore username and password.
25/01/2025 22:35:38 WARNING: Password changed. Remember to update the password in the Wazuh dashboard, Wazuh server, and Filebeat nodes if necessary, and restart the services
```

Restart the services to take effect

```bash
sudo systemctl restart filebeat
sudo systemctl restart wazuh-manager.service
sudo systemctl restart wazuh-indexer
```
Login to the Wazuh Dashboard with the new password you 

![WazuhDashboard]({{site.url }}/assets/posts/wazuh/WazuhDashboard.png)


## 3. Deploying the agent

Once the server is up and running, the final process is to Deploy the agent to machine.
Here I will be deploying the agent to a windows machine.

-   On the dashboard, Select the option Deploy New Agent. 
-   Select your operating system, here I will be using windows 
-   Enter the server address
-   Enter the name of your PC, if nothing is specified it will use the hostname as the agent name.
-   Scroll down and you will find the command to run in powershell to download and activate the agent.


### Installing on the target Machine

On the target machine, open powershell and run the command and it will install the agent automatically

```pwsh
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.10.1-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='<IP ADDRESS OF SERVER>' WAZUH_AGENT_NAME='<AGENT NAME>' 
```

After the installation is done, start the agent 

```pwsh
Get-Service WazuhSvc | Start-Service
```


Please allow a few minutes for the agent to register. Afterward, return to the Wazuh Dashboard, and you should see the agent listed as active.
That's all. You have successfully deployed the Wazuh Server and installed the agent on the machine.


![WazuhAgentScreen]({{site.url }}/assets/posts/wazuh/AgentScreen.png)





