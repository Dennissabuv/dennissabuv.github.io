---
title: External Network Monitoring with Healthchecks.io on (Raspberry Pi)
categories: [ubuntu]
tags: [homeserver, firewall]
---

## Introduction
This guide explains how to securely run a scheduled Healthchecks.io cron job on a Raspberry Pi running Ubuntu Server, deployed in a DMZ network segment.

The primary motivation behind using Healthchecks.io is to ensure external uptime monitoring of my home network. Since I do not have a backup internet connection, any failure of the primary connection would leave me without visibility or alerts—especially when I am away from home. By configuring a periodic "heartbeat" from the Raspberry Pi to Healthchecks.io, I can be notified if the connection drops or if the Pi becomes unreachable.

### To enhance security, we will:

- Use a dedicated service account with least privileges
- Avoid running cron jobs as root
- Restrict file access with proper ownership and permissions
- Use a simple, reliable script via cron




## 1. Healthchecks.io Setup
- Create a free account at [HealthChecks.Io](https://healthchecks.io.)
- Create a new check, with the preffered name  and select the desired schedule
- Optionally, configure integrations like Pushover, Slack, or Telegram for alerts.
- Copy the generated ping URL (e.g., https://hc-ping.com/YOUR-UUID).please keep the URL safe as anyone with the URL can trigger check

## 2. Create a System User with Least Privileges

I wanted to create a service account with the least privilege to run the cron jobs as considering security in my mind.

``` bash
sudo adduser --system --no-create-home --disabled-login -gecos "Service account for CRON Job - Healthcheck IO" sa_netmon
```

#### Explanation :

| Option             | Purpose                                                     |
| ------------------ | ----------------------------------------------------------- |
| `--system`         | Marks it as a system user (UID < 1000, no login by default) |
| `--no-create-home` | Prevents the creation of home directory, /home/sa_netmon    |
| `--disabled-login` | Disables interactive shell logins                           |
| `--gecos`          | Added a deescription for clarity and documentation          |
____________________________________________________________________________________

you may read more > [Ubuntu Manual Page](https://manpages.ubuntu.com/manpages/questing/en/man8/adduser.8.html)

Once you have created the account, verify the created account with
```bash
getent passwd sa_netmon
```
(Optional)

I will also be creating a group and adding the user to the group and then provide the group access to the directory where the scripts are stored in

```
sudo groupadd netmon
#Check the group existence
getent group netmon
#Adding the user to the group netmon
sudo usermod -aG netmon sa_netmon
```

### 3 Test the comand you want to run as Service Account

1. You may manually test the command by initiating a ping to the URL using the service account

```bash
sudo -u sa_netmon curl -fsS --retry 3 https://hc-ping.com/YOUR-UUID-HERE

```
The Curl has been used with few options, please see below for breakdown

| Option      | Meaning                                                                                             |
| ----------- | --------------------------------------------------------------------------------------------------- |
| `-f`        | **Fail silently on HTTP errors** (like 404 or 500).                                                 |
| `-s`        | **Silent mode** – hides progress meter and errors.                                                  |
| `-S`        | **Show errors** (but only if `-s` is used). Shows message only on failure.                          |
| `--retry 3` | **Retry up to 3 times** if the request fails due to a transient issue (e.g. network down, timeout). |


### 4. Secure Script Location

I will be creating a directory under :

```
/opt/netmon-scripts/
#Script Would be Named as ─ > net-check.sh
```

```bash
sudo mkdir --p /opt/netmon-scripts
```
#### Assigning the OWnership and Permissions


```bash
sudo chown root:netmon /opt/netmon-scripts
sudo chmod 750 /opt/netmon-scripts
sudo chmod g+s /opt/netmon-scripts
```
#### Explained
* root:netmon
    - User Owner: root (the user who owns the directory)
    - Group Owner: netmon (The group who owns the directory)
* Permissions 750
    - Owner (root) → rwx (can read, write, enter)
    - Group (netmon) → r-x (can read and enter, but cannot write)
    - Others → --- (no access)
* g+s (set-gid)
    - Any new files or sub directory created inside will automatically get group netmon


The way I wanted to do it in this manner was, the service account will not be able to run or create any other scripts or make any changes to the file



### 5 Creating the script with curl


```bash
sudo nano /opt/netmon-scripts/net-check.sh
```
Add the script into the file.
Make sure that you include all the bin bash

```bash
#!/bin/bash
curl -fsS --retry 3 https://hc-ping.com/123-456-abc-defh-099
```

you can test the script by running as the service account as below:
```bash
sudo -u sa_netmon /opt/netmon-scripts/net-check.sh
```

** if the check fails to run ensure that the permissiosn are properly applied or add the permission to execute the script
```bash
sudo chmod 750 /opt/netmon-scripts/net-check.sh
```

### 6 Setting up the CRON job for the Service Account

here I plan to run the CRON Job every 5 minutes

* Open the service account's crontab

```bash
sudo crontab -u sa_netmon -e

#When you open for the first time, it will ask select the editor of your choice.
#Enter the choice as the as provided

## ADD THE CRON JOB to the bottom

*/5 * * * * /opt/netvigil-scripts/net-check.sh

```
### 7  Viewing the logs


Cron messages are logged via rsyslog into /var/log/syslog. To see only cron entries, you can filter:

```bash
# Show recent cron entries
sudo grep CRON /var/log/syslog | tail -n 50
```

you can also view the real time by

```bash
sudo tail -f /var/log/syslog | grep --line-buffered CRON

```

This setup gives you a secure, least-privileged monitoring cron job that cleanly reports to Healthchecks.io.
