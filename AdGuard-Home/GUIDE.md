# AdGuard Home DNS Sinkhole

This docker container exposes ports 53 (DNS), 80 (HTTP), and 3000 (Web GUI initial setup).  
  
Once the container is spun up, navigate to x.x.x.x:3000 to log in, where x.x.x.x is the IP address specified in the configuration file.   

## Resources:
* Docker macvlan networks with IPV6
  * [Reddit - Docker Deployments and IPv6](https://www.reddit.com/r/ipv6/comments/1alpzmb/docker_deployments_and_ipv6_how_do_you_do_it/)
  * [Reddit - Change Configuration of Existing Docker Network](https://stackoverflow.com/questions/64596780/how-to-change-configuration-of-existing-docker-network)
    
* Enabling macvlan container to host communication
  * [Reddit - How to Create Macvlan Interfaces with systemd-networkd](https://www.reddit.com/r/systemd/comments/k17jcf/how_to_create_macvlan_interfaces_with/)
  * [Systemd-networkd and Macvlan Interfaces](https://major.io/p/systemd-networkd-and-macvlan-interfaces/)
  * Old, corrected by the two above: [Reddit - Docker Macvlan the Correct Way](https://www.reddit.com/r/selfhosted/comments/rzbz6h/docker_macvlan_the_correct_way/)

   
## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/adguardhome
  ```

Create two sub-directories that will be needed to store data for the service:  

  ```
  mkdir /opt/adguardhome/data
  mkdir /opt/adguardhome/conf
  ```  

Change into the `/opt/adguardhome` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now, on any device on the home network, navigate to the web GUI on your browser at [IP-ADDRESS]:3000 to log in.  

**To test AdGuard Home's functionality:** 
* Point any client device's IPv4 and IPv6 DNS to the IP address chosen for the container.
* Navigate to some websites, and check the AdGuard Home Query log to see the requests.
* You should see those requests being reflected. AdGuard Home is now serving as the DNS resolver for your client device. 

***Note:**  
While other clients can use this AdGuard Home service as their DNS resolver, the host machine (on which the docker container is running) cannot. This is because docker hosts cannot communicate with docker containers on macvlan networks.  
  
The next section will cover how to resolve this and have the host machine be able to use the AdGuard Home docker container as its DNS nameserver, as well.

## Docker Container-to-Host Maclvan Bridge  

First, change your home server's DNS configurations to use the AdGuard Home container as its DNS. Open the following configuration file:    
  
  ```
  sudo nano /etc/systemd/resolved.conf
  ```

Uncomment the following two lines, and make these changes:  

`#DNS=` --> `DNS=x.x.x.x xx:xx:xx:xx:xx:xx:xx:xx`  
`#DNSStubListener` --> `DNSStubListener=no`  

Where **_x.x.x.x_** is the IPv4 address of the AdGuard Home DNS Server specified in the docker compose file, and **_xx:xx:xx:xx:xx:xx:xx:xx_** is optionally added if the IPv6 address if enabled.  

Restart the daemon:  

  ```
  sudo systemctl reload-or-restart systemd-resolved
  ```

You should see your host machine interface now using the set DNS server with these commands:  
  ```
  resolvectl status
  sudo cat /etc/resolv.conf
  ```

However, running a `dig google.com` should return an error, because the host cannot talk to the DNS server in the macvlan docker network.  

To demonstrate this more clearly, trying to `ping x.x.x.x` (the address of the AdGuard Home DNS service) will also time out.  






