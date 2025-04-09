---
title: Managing Rule Order with PFBlockerNG on pfSense
categories: [pfSense]
tags: [security, homeserver, firewall]
---

## Introduction
When using **PFBlockerNG** alongside custom firewall rules in **pfSense**, automatic updates can disrupt your rule ordering. This guide explains how to maintain control over rule priority while keeping PFBlockerNG functional.

---

## IPv4 Configuration
To prevent PFBlockerNG from reordering rules while maintaining updates:
1. **Modify the rule description** to ensure persistence.
2. Set the **Action** to **`Alias Deny`** (instead of default blocking behaviors).

![pfBlockerIPV4]({{site.url }}/assets/posts/pfsense/pfblocker-rulechange/pfblockeripv4.png)

## Interface Rules Setup
I have not auto applied the rules to all my interface, have only selected on interface and kept the other interface as is.
### Key Adjustments:
- **Selective Interface Application**: Apply rules only to specific interfaces (avoid "Auto" for all).  
- **Floating Rules**: Disabled to prevent conflicts.  
- **Firewall Settings**:  
  - **`Firewall 'Auto' Rule Suffix`**: Enabled (`auto rule`).  
  - **`Kill States`**: Enabled.  

## DNSBL Configuration
For **DNS-based blocking (DNSBL)**:
- **Mode**: Unbound (recommended for stability).  
- **Behavior**:  
  - **`Deny Outbound`** for blocked IPs.  
  - **Floating Rules**: Disabled.  
- **Group Settings**:  
  - **`DNSBL Groups`** → Set to **`Unbound`**.  

---
## Firewall Aliases & Rule Creation
### Step 1: Identify Aliases
Navigate to **Firewall > Aliases > All** and note the alias names you want to enforce.

### Step 2: Add Custom Rules
1. **Select Interfaces**: Apply rules only to relevant interfaces.  
2. **Configure Block Rules**:  
   - **Action**: `Reject`  
   - **Source**: `*Any`  
   - **Destination**: `Address or Alias` → Select your target alias.  
   - **Description**: **`DO NOT REMOVE`**

![pfBlockerIPV4]({{site.url }}/assets/posts/pfsense/pfblocker-rulechange/pfblockerrule.png)
---

## Final Steps
- **Reload Firewall**: Apply changes via **Diagnostics > Reload Firewall**.  
- **Monitor Logs**: Verify behavior under **Status > System Logs > Firewall**.  

By following these steps, you ensure **PFBlockerNG updates proceed without disrupting your custom rule hierarchy**.