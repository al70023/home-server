# Wireguard VPN Tunnel

This docker container exposes ports 51820/udp, and 51821/tcp (Web GUI).  

This setup requires using a Dynamic DNS service, to have a public address for your home network to point the VPN to. This will also require port forwarding 51820 on your home network router to the host IP address, so requests can reach your server running Wireguard.  
  
Once the container is spun up, navigate to x.x.x.x:51821 to log in, where x.x.x.x is the IP address specified in the configuration file.   

Later, once Nginx Proxy Manager is configured, go back and remove the line that exposes port 51821, as the web GUI will only be accessible through the Proxy. Also, you will remove the default network created with this container, and instead attach this to the Nginx proxy docker network.  


## Resources:
* Setting Up Wireguard VPN with PC clients
  * [The $0 Home Server](https://youtu.be/IuRWqzfX1ik?si=pfoNZxvBvQwZNPMw&t=508)
  * [PiVPN / WireGuard Complete Setup 2022](https://www.crosstalksolutions.com/pivpn-wireguard-complete-setup-2022/#:~:text=In%20the%20WireGuard%20app%2C%20click,see%20devices%20on%20your%20network)
  * [Reddit - Wireguard On-Demand with Windows](https://www.reddit.com/r/WireGuard/comments/188a5ca/wireguard_ondemand_with_windows_my_solution/)
    
* Port Forwarding
  * [Redit - PortForwarding Help : r/ATT](https://www.reddit.com/r/ATT/comments/1aqffq8/portforwarding_help/)   


## Duck DNS Setup:

A Dynamic DNS is required if you need to find your home network and you don’t have a static IP address. Most ISPs don’t provide a static IP address or change your IP address periodically. If you don’t know if you have a static IP address, you can set up a DDNS regardless.

A DDNS is a service that gives you a hostname (a web address) that always points to your home network’s public IP address. There are many DDNS services, but this setup will use Duck DNS, which is free, permanent, and simple to use.  

Go to [duckdns.org](https://www.duckdns.org/) and sign in using your preferred method. Then, create a domain with any name that you like, as long as it isn’t already taken.  

![image](https://github.com/user-attachments/assets/66a0f908-1178-4448-a2d9-64ec0b185d57)  

Then, click on install, go the the Linux Cron section, and follow the instructions you see on the page.  

![image](https://github.com/user-attachments/assets/ee91c4fc-5315-4aff-92fd-70f34e924191)  


## Port Forwarding:

In order for Wireguard to work, you will need to forward port 51820 in your router’s web interface.  

Make sure to select the IP of your server as the LAN IP Address, 51820 as the port on both WAN and LAN and use UDP as the protocol.  

***Note:**  
* If you configured the `docker_bridge` macvlan interface in the AdGuard Home DNS setup, your router might be confusing your host server's internal IP with the IP of that virtual interface.
* If you are not able to select your server's IP address in your router settings to port forward, and it is only selecting the maclvan interface IP address, then temporarily disable that virtual interface:

  ```
  sudo ip link set docker_bridge down
  ```

* You can test that this worked by running `sudo networkctl` and seeing the interface is down. Pinging the AdGuard Home IP address should ensure that the interface is down.  

* Now, you may set up the port forwarding on your router to the host server, and then bring the interface back up:  

  ```
  sudo ip link set docker_bridge up
  ```

* Confirm it is up again by inspecting the output of `sudo networkctl` and pinging the AdGuard Home IP address once more.  

   
## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/wireguard
  ```  

Change into the `/opt/wireguard` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now spin up the container with `docker compose up -d`, and on any device on the home network, navigate to the web GUI on your browser at [IP-ADDRESS]:51821 to log in.  

**To test Wireguard VPN's functionality:** 
* Set up a client, such as a smartphone or PC, with the Wireguard configuration.
* Make sure you are not connected to the home network.
* Browse the internet, and try accessing an internal service, such as one on the `192.168` network. You should be able to access it, and see the requests in AdGuard Home, as the VPN uses the local DNS.
* Also, set up a split tunnel on your phone with On-Demand, excluding your home network, and enabling on wifi and data, to always be connected to the local DNS.
  * For split tunnels, specify allowed IPs to be only the home network subnet `192.168.10.0/24` and optionally, the IPv6 subnet, found on the router settings ending in `::/64`.

