# AdGuard Home DNS Sinkhole

## Resources:
* Docker macvlan networks with IPV6
  * [Reddit - Docker Deployments and IPv6](https://www.reddit.com/r/ipv6/comments/1alpzmb/docker_deployments_and_ipv6_how_do_you_do_it/)
  * [Reddit - Change Configuration of Existing Docker Network](https://stackoverflow.com/questions/64596780/how-to-change-configuration-of-existing-docker-network)
    
* Enabling macvlan container to host communication
  * [Reddit - How to Create Macvlan Interfaces with systemd-networkd](https://www.reddit.com/r/systemd/comments/k17jcf/how_to_create_macvlan_interfaces_with/)
  * [Systemd-networkd and Macvlan Interfaces](https://major.io/p/systemd-networkd-and-macvlan-interfaces/)
  * Old, corrected by the two above: [Reddit - Docker Macvlan the Correct Way](https://www.reddit.com/r/selfhosted/comments/rzbz6h/docker_macvlan_the_correct_way/)

   
## Guide:  

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
