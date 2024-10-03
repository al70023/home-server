# Wireguard VPN Tunnel

This docker container exposes ports 51820/udp, and 51821/tcp (Web GUI).  
  
Once the container is spun up, navigate to x.x.x.x:51821 to log in, where x.x.x.x is the IP address specified in the configuration file.   

Later, once Nginx Proxy Manager is configured, go back and remove the line that exposes port 51821, as the web GUI will only be accessible through the Proxy. Also, you will remove the default network created with this container, and instead attach this to the Nginx proxy docker network.  


## Resources:
* Setting Up Wireguard VPN with PC clients
  * [PiVPN / WireGuard Complete Setup 2022](https://www.crosstalksolutions.com/pivpn-wireguard-complete-setup-2022/#:~:text=In%20the%20WireGuard%20app%2C%20click,see%20devices%20on%20your%20network)
  * [Reddit - Wireguard On-Demand with Windows](https://www.reddit.com/r/WireGuard/comments/188a5ca/wireguard_ondemand_with_windows_my_solution/)

   
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

Now, on any device on the home network, navigate to the web GUI on your browser at [IP-ADDRESS]:51821 to log in.  

**To test Wireguard VPN's functionality:** 
* Set up a client, such as a smartphone or PC, with the Wireguard configuration.
* Make sure you are not connected to the home network.
* Browse the internet, and try accessing an internal service, such as one on the `192.168` network. You should be able to access it, and see the requests in AdGuard Home, as the VPN uses the local DNS.
* Also, set up a split tunnel on your phone with On-Demand, excluding your home network, and enabling on wifi and data, to always be connected to the local DNS.
  * For split tunnels, specify allowed IPs to be only the home network subnet `192.168.10.0/24` and optionally, the IPv6 subnet, found on the router settings ending in `::/64`.

