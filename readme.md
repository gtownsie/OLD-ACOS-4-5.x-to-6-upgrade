- [Upgrading to ACOS 6.x.x](#upgrading-to-acos-6xx)
  - [Overview](#overview)
  - [Purpose](#purpose)
  - [General Guidelines](#general-guidelines)
  - [Unsupported Hardware and Features](#unsupported-hardware-and-features)
- [Prerequisites](#prerequisites)
- [Upgrade Requirements](#upgrade-requirements)
  - [System Requirement](#system-requirement)
  - [System Partitions](#system-partitions)
  - [Review Boot Order](#review-boot-order)
    - [Upgrade Process](#upgrade-process)
  - [Download Software Image](#download-software-image)
  - [Perform a Backup](#perform-a-backup)
    - [CLI Configuration Backup](#cli-configuration-backup)
    - [GUI Configuration Backup](#gui-configuration-backup)
  - [Pre-Upgrade Tasks](#pre-upgrade-tasks)
    - [Upgrade Preparation Checklist](#upgrade-preparation-checklist)
- [Upgrade Instructions](#upgrade-instructions)
  - [CLI Configuration](#cli-configuration)
  - [GUI Configuration](#gui-configuration)
  - [Post-Upgrade Tasks](#post-upgrade-tasks)
- [Rollback Upgrade](#rollback-upgrade)
- [Restore from a Backup](#restore-from-a-backup)
  - [Key Considerations for System Restore](#key-considerations-for-system-restore)
    - [System Memory](#system-memory)
    - [FTA versus Non-FTA](#fta-versus-non-fta)
    - [L3V Partitions](#l3v-partitions)
    - [Port Splitting](#port-splitting)
    - [Port Mapping](#port-mapping)
    - [Restore Example](#restore-example)
    - [CLI Configuration](#cli-configuration-1)
  - [GUI Configuration](#gui-configuration-1)



# Upgrading to ACOS 6.x.x 

## Overview 

The Thunder device is provided with preinstalled ACOS software along with an ADC license. When you power ON the device, it boots up with the preinstalled software. To access the latest new features and software fixes as they become available, you must upgrade the ACOS software. 

If you are a new ACOS user, check the following documentation on the A10 Documentation Site:  
- For instructions on installing new hardware, see Installation Guide for Thunder Physical Appliance. 
- For instruction on installing vThunder, see Installation Guide for Thunder Virtual Appliance. 
- For instructions on installing cThunder, see Installation Guide for Thunder Container. 
- For instructions on installing ACOS on Bare Metal, see Installation Guide for Bare Metal. 
- For instructions on acquiring a product license, see Global Licensing Manager.  
- For initial configuration instructions and quick processes handbook, see Quick Start Guide. 

## Purpose 

This guide provides detailed instructions for upgrading from ACOS 4.x or 5.x to the latest version of 6.x. It includes information on pre-upgrade preparations, the upgrade procedure, post-upgrade tasks, troubleshooting tips, and additional resources. 

The following topics are covered:  

- General Guidelines
- Prerequisites
- Pre-Upgrade Tasks 
- Upgrade Instructions 
- Post-Upgrade Tasks 
- Rollback Upgrade 

## General Guidelines 

Consider the following recommendations before upgrading the ACOS device: 

- Test the upgrade procedure in a non-production environment to ensure its effectiveness. 

- ACOS device is upgraded by copying the software image to your device or other system on your local network and then upgrading the device using the CLI or GUI instructions. 

- Regardless of whether you have an ADC, CGN, or TPS, a single software image is used to upgrade your ACOS device. However, ensure that the correct product license is obtained and activated.  

> NOTE:	For TPS upgrade instructions, see TPS Upgrade Guide. 

- During the reboot, the system performs a full reset and will be offline. The actual duration may vary depending on the system parameters.  

## Unsupported Hardware and Features

- The 3rd Generation Hardware Platforms cannot be upgraded to ACOS 6.x version. For more information, see Hardware Platforms Support.  

- The Web Application Firewall (WAF) is no longer supported starting from the ACOS 6.x release. Hence, all WAF configurations will be removed after the upgrade. For more information, see Web Application Firewall Changes. 

# Prerequisites 

This section outlines essential information that you should know before proceeding with the upgrade process.  

 

Table 1 : Prerequisite Tasks 

| Tasks | Refer |
|:--------|----------|
Check the platform compatibility versus the supported release version. | Hardware Platforms Support 
Check the SKUs or product licenses availability. | Hardware Product Licenses 
Check the storage and memory requirement. | System Requirement 
Carefully review the new features, known issues, and changes to default behavior. | Documentation Site 
Understand how ACOS selects the boot order. | Review Boot Order 
Understand what the ACOS partitions and how to take a backup. | System Partitions 
Check the instructions for taking a system backup. | Perform a Backup 
Download the ACOS software image. | Download Software Image 

> NOTE: 	  

- Schedule a maintenance window for the upgrade, considering the potential downtime required. Communicate this schedule to relevant stakeholders. 

- Inform all users about the scheduled downtime and ensure they save any unsaved work or log out of the system before the upgrade begins. 

# Upgrade Requirements
## System Requirement 

The system requirements for ACOS software include the following: 

- For ACOS 6.x releases, the minimum disk space requirement is 8 GB.  
- For vThunder and Thunder devices, the minimum memory requirement is 8 GB. 

## System Partitions 

Each ACOS device contains one shared partition. By default, this is the only partition on the device and cannot be deleted. If there are no additional partitions on the device, all configuration changes occur in the shared partition. 

You can save the configuration of these partitions to either the default startup-config, the current, or a new configuration profile. To save the partition configuration, use the write memory command. 

Depending on the configuration profile and the partition being saved to, the following summarizes the write memory command usage: 

 

| Command                               | Descriptions |
|-----------------------------------------------------|--------------|
| `write memory` | Save the running configuration to the startup-config or the current profile in the current partition. |
|`write memory all-partitions` | Save the running configuration to their respective startup-config or their current profiles of all partitions.  
|`write memory <profile-name>`|Save the running configuration to the new profile in the current partition. |
`write memory <profile-name> all-partitions` | Save the running configuration to the new profile of all partitions. |

## Review Boot Order 

This section describes general guidelines on how ACOS selects the boot image. 

Each ACOS device contains multiple locations where software images can be placed. The _Upgrade Process_ table provides an overview of the general upgrade process. 

- When you load a new image onto the ACOS device, you can select the image device (disk or CF) and the area (primary or secondary) on the device.  

- When you power ON or reboot the ACOS device, it always attempts to boot from the disk, using the image area specified in the configuration (primary disk, by default). If a disk fails, the device attempts to boot from the same image area on the backup disk (if applicable to the device model). 

You need to change the boot order only when you plan to upload the new image into an image area other than the first image area the ACOS device uses when it boots (primary disk). To change the boot order, use the bootimage command.  

> NOTE:	A10 Networks recommends installing the new image into the inactive disk image area, either primary or secondary, while retaining the old image in the other area. This helps to restore the system in case a downgrade is necessary or if an issue occurs while rebooting the new image.  

### Upgrade Process

|System|Partition 1|Upgrade|Partition 2|
|----|:----:|:------:|:----:|
|New System|Acitve||Inactive
|1st Upgrade|Inactive|-->|Active
|2nd Upgrade|Active|-->|Inactive
|Next Upgrade|Inactive|-->|Active
|Next Upgrade|Active|-->|Inactive


## Download Software Image 

A10 Networks has two device types, FTA and non-FTA.  All vThunder devices will use the non-FTA version and depending on the hardware type will determin the correct image.  To determine if your device has an FTA, login to the device and run the following command:
```
ACOS# show hardware | inc FPGA
```
If a response is shown then the device had and FTA.
```
FPGA       : 4 instance(s) present
```
if the device does not have an FTA, no response to the ``show hardware`` command is displayed

Log in to A10 Networks Support using the GLM credential and download the ACOS upgrade package as specified below:  

- For FTA enabled platforms, use the image with the file name: 
```
ACOS_FTA_<version>.upg
```
- For Non-FTA enabled platforms (including vThunder), use the image with the file name: 
```
ACOS_non_FTA_<version>.upg
```
## Perform a Backup 

It's essential to perform a complete backup of your data, including configuration settings, databases, and any customizations. This backup will prove invaluable in case of unexpected issues during the upgrade and you want to restore it. For information about restoring a backup, see Restore from a Backup.  

This section provides examples of how to back up your system. 

### CLI Configuration Backup 

It is recommended to backup the system and the log files prior to upgrading the software.  
- The following example creates a backup of the system (startup-config file, aFleX scripts, and SSL certificates and keys) on a remote server using SCP:

```
ACOS(config)# backup system scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backupfile.tar.gz
```

- The following example creates a daily backup of the log entries in the syslog buffer. The connection to the remote server will be established using SCP on the management interface (use-mgmt-port).  

```
ACOS(config)# backup log period 1 use-mgmt-port scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backuplog.tar.gz
```

### GUI Configuration Backup

1. Log in to ACOS Web GUI using your credentials. 
1. Navigate to System >> Maintenance >> Backup.  
   >  == Add screenshot? 

## Pre-Upgrade Tasks 

Before upgrading ACOS software, you must perform some basic checks. Keep the below information handy to ensure a seamless upgrade.  


### Upgrade Preparation Checklist 


- [ ] Verify platform compatability:
  ```
   ACOS(config)#show hardware | inc Gateway
  ``` 
    Validate the platform is supported on version 6.x
     - vThunder:
        ```
        Thunder Series Unified Application Service Gateway vThunder
        ```
    - Hardware:
      ``` 
      Thunder Series Unified Application Service Gateway TH5840S
      ```
- [ ] Check the current software version
  ```
  ACOS>show version | inc ACOS
  ``````
  Validate that the current version is 4.x or later.
  ```
  64-bit Advanced Core OS (ACOS) version 5.2.1-p5, build 114 (Jul-14-2022,05:11)
  ```

- [ ] Check the current system disk space and verify minimum disk requriements 
     ```ACOS(config)#show disk
        ACOS(config)#show disk
        Total(MB)    Used(MB)       Free(MB)       Usage
        ---------------------------------------------------
        20480          10421          10058          50%
        Hard Disk Primary Status : OK
        
  
- [ ] Check Memory: 
  
  ```
  ACOS(config)#show memory | inc Memory
  ```


  Verify minimum memory requriements, from first column:
  ```
  Memory:  8127392      4742619     3384773   58.30%
  ```

- [ ] Check the system boot order to determine new destination:
  ```
  ACOS(config)#show bootimage | inc *
  ``` 
  This will display the current Default boot location
  ```
  Hard Disk primary         5.2.1-p5.114 (*)
  ```
- [ ] Save all primary, secondary, and partition configurations
  ```
  write memory all-partitions 
  Building configuration...
  Write configuration to default primary startup-config
  Write configuration to profile "pri_default" on partition GSLB 
  [OK]
  ```
- [ ] Backup the system configuration
  ```
  ACOS(config)# backup system scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backupfile.tar.gz
  ```
- [ ] Backup system log files
  ```
  ACOS(config)# backup log period 1 use-mgmt-port scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backuplog.tar.gz
  ```
 >See Also:  
 For detailed information on all the commands, see ***Command Line Interface Reference***.

# Upgrade Instructions 

This section describes the upgrade instructions using CLI and GUI. The upgrade instruction provided in this section applies to FTA platforms, non-FTA platforms, and non-aVCS environments.  

## CLI Configuration 

1. Complete Upgrade Preparation Checklist
1. Upgrade the ACOS device to the inactve partition.  

   - If the primary hard disk is active upgrade the secondary hard disk: 
   ```
   ACOS-5-x(config)# upgrade hd sec scp://2.2.2.2/images/ACOS_<version>.upg
   ```
   
      > Note:  Use the approprate FTA or non-FTA ACOS version identified in the Upgrade Preparation Checklist

     - If the secondary hard disk is active upgrade the primary hard disk:
    
    ```
    ACOS-5-x(config)# upgrade hd pri scp://2.2.2.2/images/ACOS_<version>.upg
    ```
    > Note:  Use the approprate FTA or non-FTA ACOS version identified in the Upgrade Preparation Checklist   

1. You will be prompted to reboot your ACOS device
    
   ==Choose "NO"==
  
1. You will be prompted to reboot your ACOS device. 

1. Press yes to reboot and bring up the upgraded ACOS software.  
  
> Allow up to five minutes for the reboot to complete. (The typical reboot time is 2-3 minutes.) 

6. Import the required license and reboot again.  
  The upgrade process is completed successfully.  

## GUI Configuration 

1. Log in to ACOS Web GUI using your credentials. 
1. Navigate to System >> Maintenance >> Upgrade.  
1. On the Upgrade page, click ? to open the Online Help. 

The Online Help provides complete details on upgrade and rollback instructions.  

## Post-Upgrade Tasks 

After performing upgrade, it is important to perform some basic post-upgrade checks.  

Table 3 : Post-Upgrade Checklist 

Tasks|Command or Action
|----|----|
Verify that the upgrade was successfully|ACOS>`show version` 
Verify the required license is imported successfully|ACOS(config)#`show license-info`
Verify if the saved configuration from all the partitions are loaded successfully|ACOS(config)#`startup-config [all \| all-partitions \| partition \| profile]`
Configure any new features or settings introduced in the latest release|Refer New Features and Enhancements guide from the documentation site.  
Conduct thorough functional testing to ensure that all core features and functionalities work as expected in the latest version|NA 

 

# Rollback Upgrade 

In case the upgrade encounters significant issues or if it fails, have a rollback plan ready to revert to the previous version. The rollback for ACOS device is similar to the upgrade process.  

 

**Table 4 : Rollback Tasks**

Tasks
Refer 
Carefully review the restoring the system backup information.
Key Considerations for System Restore 

Download your current version ACOS software image.
Download Software
Perform the upgrade instructions
Upgrade Instructions  
Restore the backed up configurations
Restore Example 
Perform the post-upgrade tasks
Post-Upgrade Tasks 

 

# Restore from a Backup 

You can use a saved backup to restore your current system, for example, when upgrading the devices in your network to the newer A10 Thunder Series devices.  

## Key Considerations for System Restore 

### System Memory 

If the current device has insufficient memory compared to the backup device (for example, 16 GB on the current device compared to 32 GB on the previous device), this can adversely affect system performance.  

### FTA versus Non-FTA 

When restoring from an FTA device to a non-FTA device, some commands may become unavailable after the restore operation. These commands are lost and cannot be restored. 

### L3V Partitions 

L3v partitions and their configurations are restored. However, if you are restoring to a device that supports a fewer number of partitions (for example, 32) than you had configured from the backup device (for example, 64) any partitions and corresponding configuration beyond 32 will be lost. 

### Port Splitting 

If you are restoring between devices with different 40 GB port splitting configurations, see Table 5. 

 

Table 5 :  Restore Behavior for Port Splitting Combinations  

Backup Device 

Current Device 

Behavior During the Restore Operation 

Port splitting disabled or enabled. 

Port splitting disabled or enabled. 

Allow user to perform port mapping (See Port Mapping.) 

Port splitting enabled. 

Port splitting disabled. 

Ask the user if they want to perform port mapping. If yes, enable port splitting, reboot the device, and then perform the restore operation again, where port mapping will be enabled. 

Port splitting disabled. 

Port splitting enabled. 

Exit the restore operation. The user will have to perform a system-reset or disable port splitting, reboot the system, and then perform the restore operation again. 

 

### Port Mapping 

When restoring from a device that has a different number of ports, or even the same number of ports, you can map the port number from the previous configuration to a new port number (or same port number) in the new configuration.  

In cases where the original number of ports is greater than the number of ports on the new system, some configurations may be lost. 

If you choose to skip port mapping (see the example below), then the original port numbers and configurations are preserved. If the original device had ports 1-10 configured, and the new device only has ports 1-8, and you skip port mapping, then ports 9 and 10 are lost. If you choose port mapping, you can decide which 8 out of the original 10 ports you want to preserve during the port mapping process. 

### Restore Example 

This section provides an example of a restore operation: 

- The backup is restored from version 4.1.1-P1 to 4.1.1-P2.  
- The system memory on the original device is 8 GB, but is 16GB on the new device. 
- The number of interfaces on the original device is 10, but the new device has 12. 

### CLI Configuration 

See the highlighted lines in the following example output along with the corresponding comments that are marked with “<--“characters: 

  
```
ACOS(config)# restore use-mgmt-port scp://root@192.168.2.2/root/user1/backup1 
Password []?  

A10 Product: 
| Object | Backup device | Current device 
|----------------|-------------------------------|--------------------- 
| Device | TH1030 | TH3030 
|Image version | 4.1.1-P1 | 4.1.1-P2 

System memory: 
    Object               Backup device        Current device 
------------------------------------------------------------------- 
    Memory (MB)          8174                 16384 

Checking memory: OK. 

Ethernet Interfaces: 
    Object               Backup device        Current device 
------------------------------------------------------------------- 
    Total                10                    12 
    1 Gig                1-10                  1-12 

Do you want to skip port map?(Answer no if you want port mapping manually.) 
[yes/no]: no 

Please specify the Current device to Backup device port mapping 
1-10 : a valid port number in backup device. 
0    : to skip a port 
-1   : to restart port mapping. 

Current Port:    Backup device port 
Port 1 : 2 <-- port 2 on the backup device is re-numbered to 1 
Port 2 : 1 <-- port 1 on the backup device is re-numbered to 2 
Port 3  :            0 
Port 4  :            0 
Port 5  :            0 
Port 6  :            0 
Port 7  :            0 
Port 8  :            0 
Port 9  :            0 
Port 10 :            0 

The current startup-configuration will be replaced with the new configuration that was imported. 

Do you wish to see the diff between the updated startup-config and the original backup configuration? 

[yes/no]: yes 

Modified configuration begin with "!#" 

!Current configuration: 277 bytes 
!Configuration last updated at 05:38:18 UTC Fri Mar 17 2017 
!Configuration last saved at 05:38:19 UTC Fri Mar 17 2017 
!64-bit Advanced Core OS (ACOS) version 4.1.1-P2, build 112 (Mar-13-2017,15:41) 
! 
interface management 
  ip address 192.168.210.24 255.255.255.0 
  ip default-gateway 192.168.210.1 

!#interface management 
!#  ip address 192.168.210.24 255.255.255.0 
!#  ip default-gateway 192.168.210.1 
!#  exit-module 
! 
interface ethernet 2 
!#interface ethernet 1 <-- original port 1 is now port 2 
  exit-module 
! 
interface ethernet 1 
!#interface ethernet 2 <-- original port 2 is now port 1 
  exit-module 
! 
!#interface ethernet 3 
!#  exit-module 
! 
!#interface ethernet 4 
!#  exit-module 
! 
!#interface ethernet 5 
!#  exit-module 
! 
!#interface ethernet 6 
!#  exit-module 
! 
!#interface ethernet 7 
!#  exit-module 
! 
!#interface ethernet 8 
!#  exit-module 
! 
! 
end 

Complete the restore process? 
[yes/no]: yes 

Please wait restore to complete: . 

Restore successful. Please reboot to take effect. 
```
## GUI Configuration 

1. Log in to ACOS Web GUI using your credentials. 
1. Navigate to System >> Maintenance >> Restore.  
> ==Need to get screen shots
 