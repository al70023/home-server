# Nginx Proxy Manager

This docker container exposes port 80 (HTTP), 443 (HTTPS), and 81 (web GUI). 
  
Once the container is spun up, navigate to x.x.x.x:81 to log in, where x.x.x.x is the IP address specified in the configuration file.   

This setup also involves creating a custom bridge docker network for managing this container, as well as any other services that will be accessed through the Proxy.


## Resources:
* Explaining Proxy Ports Exposing
  * [Reddit - Nginx Proxy Manager: How do I have a container behind NPM that uses port 80?](https://www.reddit.com/r/selfhosted/comments/1apym8k/nginx_proxy_manager_how_do_i_have_a_container/)
    
* Setup
  * [Self Hosting on Your Home Server - Cloudflare + Nginx Proxy Manager](https://www.youtube.com/watch?v=GarMdDTAZJo&t=567s)


## Duck DNS Setup:

This setup will use Duck DNS again, pointing to an internal IP address. The domain name will be used to validate ownership and create SSL certificates that can be attached to proxy hosts, for encrypted connections to your services.      

Use existing subdomain that points to an internal network IP address (i.e. `192.168.x.x`).  

If you do not already have one, go to [duckdns.org](https://www.duckdns.org/) and sign in using your preferred method. Then, create a domain with any name that you like, as long as it isn’t already taken.  

Replace the IP address on this subdomain with the address of your host server on the home network, so that this domain can only be accessed internally within your home network, i.e. `192.168.10.x`.     


## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/nginxproxymanager
  ```

Create sub-directories to hold persistant storage:  

  ```
  mkdir /opt/nginxproxymanager/data
  mkdir /opt/nginxproxymanager/letsencrypt
  ```  

Change into the `/opt/nginxproxymanager` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now, on any device on the home network, navigate to the web GUI on your browser at [IP-ADDRESS]:81 to log in.  

  
