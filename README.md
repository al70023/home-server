# Getting Started
* If using laptop, make sure to disable sleep on lid close before any install.  
* Backup any data that may be lost.  
* Run benchmarks for CPU, check NIC max throughput, and check network speeds.

## Linux Download

1. Download [Ubuntu Server LTS](https://ubuntu.com/download/server)

2. Flash the ISO to a USB Stick using Balena Etcher
   * Make sure no important data is on the USB, it will be wiped
  
## Installation

Follow the prompts on screen to install Ubuntu. 
* Leave the OpenSSH Server checked to have remote access to your machine once it’s set up. 
* Make sure you uncheck “Set up this drive as an LVM group” when asked about partitioning your drive. If you were to leave that checked, it would split your drive into multiple volumes, using only part of your disk, and you would need to manually resize it later, which complicates the process.

![image](https://github.com/user-attachments/assets/b22bcdf2-74f8-4ba2-b80d-591594800049)

## First Login

After successfully logging in with created root user account, follow basic steps to harden security on the system.  

### 1. Run updates

### 2. Reserve static IP address for the machine, either through DHCP lease (router settings) or in configuration files.

   * Example setting up static IP on machine:
     
     `sudo nano /etc/netplan/50-cloud-init.yaml`
      ```
        network:  
         version: 2  
         ethernets:  
           enp5s0:                   # Name of the interface     
             dhcp4: false            # No DHCP lease, IP is set here
             addresses:  
               - 192.168.1.140/24    # machine IP adress to set 
             routes:  
               - to: default  
                 via: 192.168.1.254  # router/gateway of home network
             nameservers:            # Optional, DNS servers to set (default will be assigned by network router/gateway)
               addresses:  
                 - 192.168.1.140     # machine IP adress to set 
      ```
      `sudo netplan apply`
       
      And to confirm and see the interface with the IP:
     
      `resolvectl status`  



