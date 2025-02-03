---
title: Changing the VM-ID in Proxmox.
categories: [Proxmox]
tags: [proxmox,homeserver]
---

In Proxmox, the VMID is the unique identifier assigned to a virtual machine or container. While Proxmox does not provide a built-in feature to change a VMID directly, you can achieve this by cloning the VM or container or by manually renaming and reconfiguring the files. Here's how to do it:

## Cloning - The safest and easiest method.

#### 1 - Log in to Proxmox Web Interface: 
Access your Proxmox VE web interface.

#### 2 - Clone the VM/Container:

    - Right-click on the VM or container.
    - Select Clone.
    - In the Clone Options, set the new VMID.
    - Finish the cloning process.

Delete the Old VM once you have confirmed that the cloned VM works.

## Manual Renaming 

This method involves renaming the system files, please create a backup of the VM before you proceed.

Here in this example, I will rename the VM ID 200 of (SERVER01) to VM 110.

#### 1. Stop the VM : 
Stop the VM and shutdown from the web interface or you can use the below command and run in the shell

``` bash
#List all the VM's with their ID's
qm list 
```

once you have identified the ID of the VM

``` bash
qm stop <VMID>
# Example
# qm stop 200
```
Confirm the stop by running

``` bash
qm list | grep "<VM ID / Server Name>" 
# Example
# qm list  grep "200"
```

#### 2. Rename the Configuration file

Navigate to the location where Proxmox stores the VM configuration file to list the configuration of the VM, this is needed to identify the storage location, edit the conf file and move the configuration

``` bash
cd /etc/pve/qemu-server/
```
- List the files using, ls command and include the VM ID to just list the conf file of that VM.

```bash
ls 200.conf
```
- Rename the configuration file

```bash
mv <OLD-VMID>.conf <NEW-VMID>.conf
# Example
# mv 200.conf 110.conf
```

This will rename the configuration file, now we need to move the disk file that the VM uses
### 3. Move the Disk File

- Check the configuration file and look for the lines that shows the storage locations.
Please note, the conf file will now be in the new VM-ID

``` bash
nano <NEW-VMID>.conf
# Example
#nano 110.conf
```
Look for lines starting with scsi, virtio, or ide that reference storage locations. 

Alternatively, use grep:

``` bash
 cat 110.conf | grep "scsi*"
 # Change the scsi to the storage method you have
```
![StorageList]({{site.url }}/assets/posts/proxmox/vmidchange/storagelist.png)

``` bash
#VM Disk File Name :vm-200-disk-0.qcow2 
```
here, if you look at the picture the storage location is lv-vms, I will be looking at the mount point to navigate to the storage location to edit and move the file

you can use the find tool to search on where the disk images are located

``` bash
# Move to the user's home directory
cd ~ 
find / -name "<NAME OF THE DISK FILE>"

# Example
# find / -name "vm-200-disk-0.qcow2"
```

Here in my example the location shows 
```bash
 /mnt/data/vms/images/200/vm-200-disk-0.qcow2
```

- Moving the disk files

Navigate to the storage path and rename the disk files, note the disk file will still be in the old VM ID, the new path should be moved to new directory with new VM ID.

    - New Directory needs to be created
    - The disk image must be renamed to new one
    - Old directory needs to be removed



``` bash
cd /path/to/storagelocation
mkdir <NEW-VMID>
mv <old-vmid>*.qcow2 /newVMID/<new-vmid>*.qcow2
rmdir <OLDVMID>
# Example
# cd /mnt/data/vms/images/
# mkdir 110
# mv 200/vm-200-disk-0.qcow2 110/vm-110-disk-0.qcow2
# rmdir 200
```

#### 4. Update the Configuration File

Navigate to the conf file of the VM, 

``` bash
cd /etc/pve/qemu-server/
nano <NEW-VMID>.conf

#here in my case,
#cd /etc/pve/qemu-server/
#nano 110.conf
```

![StorageList]({{site.url }}/assets/posts/proxmox/vmidchange/disklocold.png)


Update the configuration file, with new disk location, you should rename it to match the new location

here in my case, I have renamed the location

``` bash
#OLD LOCATION
scsi0: lv-vms:200/vm-200-disk-0.qcow2,iothread=1,size=200G
#NEW LOCATION
scsi0: lv-vms:110/vm-110-disk-0.qcow2,iothread=1,size=200G
```

Save and exit (CTRL + X, then Y and ENTER).

#### 5. Start the VM

Return to the Proxmox web console and check that the VM appears with the new ID. Start the VM to confirm that everything works correctly.
Check for errors in the Proxmox task log.

you can start the VM from the shell by

```bash
qm start <NEW-VMID>
#example 
qm start 110
```

<b>Final Checks</b>

 - Ensure the VM boots properly.
 - Verify network settings, especially if using static IPs.
 - If using snapshots, update references to the new VMID.
 - If you have additional storage, please make sure to update that in the conf file.


That's it, you have renamed the VMID .


















