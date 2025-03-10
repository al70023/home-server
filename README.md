# Getting Started
* If using laptop, make sure to disable sleep on lid close before any install.  
* Backup any data that may be lost.  
* Run benchmarks for CPU, check NIC max throughput, and check network speeds.

#### Resources:
* [The $0 Home Server - Kalos Likes Computers](https://www.youtube.com/watch?v=IuRWqzfX1ik&t=306s)
  

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

#### Resources:
* [5 Steps to Secure Linux (protect from hackers) - NetworkChuck](https://www.youtube.com/watch?v=ZhMw53Ud2tY)


### 1. Run Updates and Set Auto-Updates 

Run first manual updates:
  ```
  sudo apt update
  sudo apt upgrade
  ```

Set up automatic unattended system updates:
  ```
  sudo apt install unattended-upgrades
  sudo dpkg-reconfigure --priority=low unattended-upgrades
  ```  


### 2. Set Static IP

Reserve static IP address for the server, either through DHCP lease (router settings) or in host configuration files.

Example setting up static IP on host config:
  ```   
  sudo nano /etc/netplan/50-cloud-init.yaml
  ```
     
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

  ```
  sudo netplan apply
  ```
     
And to confirm and see the interface with the IP:

  ```
  resolvectl status
  ```    


### 3. Set SSH Key Login from Client to Host Server

First create directory to store public keys on the host:  

  ```
  mkdir ~/.ssh && chmod 700 ~./ssh
  ```

Then, on a client device that will be accessing the host server, such as a Windows PC, run:  

  ```
  ssh-keygen -b 4096
  ``` 

Finally, on the Windows client, send that public key to the host server with the following command:

  ```
  scp $env:USERPROFILE/.ssh/id_rsa.pub [YOUR-USER]@[YOUR-SERVER-IP]:~/.ssh/authorized_keys
  ```  


### 4. Remove Password Logins

Edit this file:  

  ```
  sudo nano /etc/ssh/sshd_config
  ```

Find the following lines and change: 

  `PermitRootLogin yes` --> `PermitRootLogin no`  
  `PasswordAuthentication yes` --> `PasswordAuthentication no`

Finally, restart the SSH daemon:

  ```
  sudo systemctl restart ssh
  ```  


### 5. Set Up Firewall

First, check open ports on the host:

  ```
  sudo ss -tupln
  ```

Take note of any open ports, and research why they are open. Close if necessary.

Next, install ufw (Uncomplicated Firewall) to manage ports on the host system:

  ```
  sudo apt install ufw
  ```

Check the status of the firewall:

  ```
  sudo ufw status
  ```  

Now, before activating, allow port 22 to not lock yourself out of being able to SSH into the server:

  ```
  sudo ufw allow 22
  ```  

You can now enable the firewall, and should see the ports being allowed by running another `sudo ufw status` command:

  ```
  sudo ufw enable
  ```  

*****Note<sup> 1</sup>**: Default ufw behavior is**  
* Deny all incoming
* Allow all outgoing
* Deny all routed
  
  
*****Note<sup> 2</sup>**: When using Docker containers**  
    
* Docker handles ports directly, and it can circumvent the firewall rules set up through ufw.  
* For more details on how to address:  
  * [How To Use Docker with a UFW Firewall](https://www.howtogeek.com/devops/how-to-use-docker-with-a-ufw-firewall/)
  * [ufw-docker](https://github.com/chaifeng/ufw-docker)
  * [whalewall](https://github.com/capnspacehook/whalewall)



## Set Up Docker & Portainer

By using Docker containers to run services and applications, you are ensuring a layer of separation between the program environments and the host machine, as well as portability of those installations.  
  
For setting up configuration files, use Docker Compose. This means you have permanent .yaml files that you can easily edit use to spin up or bring down docker containers.  
  
Finally, use Portainer as a web GUI to easily view all things Docker-related running on the host server, including containers, stacks, and networks.  

#### Resources:
* [Portainer Official Install](https://docs.portainer.io/start/install-ce/server/docker/linux)  


To install docker, run:

  ```
  sudo apt install docker.io
  ```

Docker Compose is included within the Docker Engine, so no separate install is necessary.  

To install Portainer, first create the volume that Portainer Server will use to store its database:  

  ```
  docker volume create portainer_data
  ```

Then, download and install the Portainer Server container:  

  ```
  docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.21.2
  ```

Portainer Server has now been installed. You can check to see whether the Portainer Server container has started by running `docker ps`:  

  ```
  root@server:~# docker ps
  CONTAINER ID   IMAGE                          COMMAND                  CREATED       STATUS      PORTS                                                                                  NAMES             
  de5b28eb2fa9   portainer/portainer-ce:2.21.2  "/portainer"             2 weeks ago   Up 9 days   0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   portainer
  ```

Now that the installation is complete, you can log into your Portainer Server instance by opening a web browser and going to:  

  ```
  https://[YOUR-HOST-IP]:9443
  ```

The host server is now ready and set up to begin running containerized services and applications.  



# Services

## 1. AdGuard Home DNS Sinkhole

This will be a local DNS Server with blocklists and an upstream resolver.   
  
This service will be implemented via a macvlan docker network, so that it can virtually appear as a seperate machine on the network, with a separate IP address from the host.  

All setup can be found [here](AdGuard-Home/GUIDE.md).  


## 2. Wireguard VPN Tunnel  

This will allow securely tunneling into the host machine from any external client not on the same home network.  

All setup can be found [here](Wireguard/GUIDE.md).

## 3. Vaultwarden Password Manager

This will be a locally-hosted solution for managing passwords for clients.   

It will not be publicly accessible, and will sync data to client machines so that they have locally stored passwords even when not on the home network.  

All setup can be found [here](Vaultwarden/GUIDE.md).  

## 4. Nginx Proxy Manager  

This will serve as local DNS rewrites for docker services running on the server.   

Users will no longer use IP-ADDRESS:PORT to navigate to a service on their browser, and instead use the URL assigned.  

This will also use Let's Encrypt SSL certificates, for HTTPS encryption on the browser when using these services.  

All setup can found [here](Nginx-Proxy-Manager/GUIDE.md).  

## 5. Portainer  

Since Portainer does not have a docker compose file, use the command line to alter its network configurations to be able to integrate it with Nginx Proxy Manager.  

Use `docker network ls` to find the default bridge network that was created with Portainer, then use `docker network rm [NETWORK-NAME]` to remove it.  

Finally, attach it to the proxy network with `docker network connect npm_proxy portainer`.  

Now, you can create a subdomain for it on Nginx Proxy Manager and access portainer securely through that subdomain, without exposing the web GUI port.  


## 6. Mealie Recipe Cookbook    

This is a locally-hosted recipe web app.

Users can create or import recipes, set meal plans, manage groups, and more.  

This will be accessible via the Nginx Proxy, to provide SSL certificates on web URL.  

All setup can be found [here](Mealie/GUIDE.md).    


## 7. Frigate NVR 

This is a self-hosted home surveillance system.    

This will be accessible via the Nginx Proxy, to provide SSL certificates on web URL.  

All setup can be found [here](Frigate/GUIDE.md).    


## 8. Uptime Kuma Health Monitor

This service can be used to monitor other services, containers, ports, and web pages.      

This will be accessible via the Nginx Proxy, to provide SSL certificates on web URL.  

All setup can be found [here](Uptime-Kuma/GUIDE.md).    


## 9. Watchtower Docker Container Automatic Updates

This service can be used to automate the updating of all other docker containers on the host, or to provide notifications for available image updates.  

There is no web GUI, and no port exposure used for this docker container.  

All setup can be found [here](Watchtower/GUIDE.md).  


## 10. Restic Docker Container Scheduled Backups

This service is crucial to automate the backups of all other docker containers on the host, via incremental snapshots.  

There is no web GUI, and no port exposure used for this docker container.  

All setup can be found [here](Restic/GUIDE.md).  

# Additional Configurations

## Setting up Network Adapter or NIC

If having trouble detecting or running on a new network interface, follow these steps to set up.  

First, checked plugged in adapters:  

  ```
  lsusb
  ```  

Check network interfaces:  

  ```
  ip a
  ifconfig
  sudo networkctl
  ```  

Check driver:  

  ```
  dmesg | grep -i [NIC-driver]
  ```  

If the interface is detected but down, manually bring it up:

  ```
  sudo ip link set dev [interface-name] up
  ```

Configure IP for the interface from DHCP lease:  

  ```
  # /etc/netplan/50-cloud-init.yaml
  
  network:
  version: 2
  ethernets:
    [interface-name]:
      dhcp4: true
  ```

And save the changes:  

  ```
  sudo netplan apply
  ```


#### Resources:
* Ethtools troubleshooting
  * [Reddit - Internet Works Very Slow in Linux](https://www.reddit.com/r/linux4noobs/comments/eyt825/internet_works_very_slow_in_linux/)
  * [Wake-on-LAN](https://wiki.archlinux.org/title/Wake-on-LAN)























