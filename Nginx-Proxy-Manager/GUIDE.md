# Nginx Proxy Manager

This docker container exposes ports 80 (HTTP), 443 (HTTPS), and 81 (web GUI). 
  
Once the container is spun up, navigate to `x.x.x.x:81` to log in, where `x.x.x.x` is the IP address specified in the configuration file.   

This setup also involves creating a custom bridge docker network for managing this container, as well as any other services that will be accessed through the Proxy.


## Resources:
* Explaining Proxy Ports Exposing
  * [Reddit - Nginx Proxy Manager: How do I have a container behind NPM that uses port 80?](https://www.reddit.com/r/selfhosted/comments/1apym8k/nginx_proxy_manager_how_do_i_have_a_container/)
    
* Setup
  * [Self Hosting on Your Home Server - Cloudflare + Nginx Proxy Manager](https://www.youtube.com/watch?v=GarMdDTAZJo&t=567s)
  * [Nginx Proxy Manager Advanced Configuration - Docker Networks](https://nginxproxymanager.com/advanced-config/)


## Duck DNS Setup:

This setup will use Duck DNS again, but pointing to an internal IP address. The domain name will be used to validate ownership and create SSL certificates that can be attached to proxy hosts, for encrypted connections to your services.      

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

Finally, create a custom bridge network that is specified in the configuration file, so that the Nginx container will be attached to this network, and not a default docker bridge:  

  ```
  docker network create npm_proxy
  ```

Now spin up the container with `docker compose up -d`, and on any device on the home network, navigate to the web GUI on your browser at `[IP-ADDRESS]:81` to log in.  


## Adding SSL Certificates:  

Once logged into the Nginx Proxy Manager web GUI, go to the "SSL Certificates" tab, select "Add SSL Certificate", and choose "Let's Encrypt" from the dropdown.  

In the **Domain Names** text box, enter two domains, separated by spaces:
1. Your Duck DNS subdomain in this format: `[YOUR-SUBDOMAIN].duckdns.org`
2. A wildcard subdomain for your Duck DNS subdomain in this format: `*.[YOUR-SUBDOMAIN].duckdns.org`  

Select "Use a DNS Challenge" and select Duck DNS from the list of providers. Paste your API token from your Duck DNS account by logging back in there, change the propogation seconds to 120, and then hit save.    

It should take a while and then give you back a success. If you receive errors, try again after 24 hours, it might take some time for your domain name to propogate.  

## Creating Proxy Hosts:  

Now, go to your dashboard, and select the "Proxy Hosts" tab.  

Select "Add Proxy Host".  

For each of the previously set up services, such as AdGuard Home, Vaultwarden, Wireguard, and Portainer, set up a subdomain to access it securely with SSL certificates.  

The format of this subdomain name should be `[SERVICE].[YOUR-SUBDOMAIN].duckdns.org`, where `[SERVICE]` can be anything you like to name the service, such as wireguard or wg for Wireguard, and `[YOUR-SUBDOMAIN]` is the one chosen for Duck DNS, which you also created SSL certificates for.  

Create a domain name, use the name of the container in **Forward Hostname/IP**, and the port of the web GUI for that container (the port does not need to be exposed in its docker compose file). Select the SSL certificate created for your subdomain, and attach it to the proxy host.  

Finally, for each of the services, ensure they are on the same docker network as Nginx Proxy Manager. For this setup, that has been `npm_proxy`.  Go back to each docker compose file for the services being run, and add the section specifying that docker network. Additionally, remove the default bridge networks created for each of those services before they were attached to this custom network.  

Use `docker network ls` to see all networks and `docker network rm [NETWORK-NAME]` to remove the ones not needed.  

Once the containers are all attached to the `npm_proxy` network, and they have a Proxy Host created in Nginx Proxy Manager, you should be able to navigate to the specified subdomain created for each service, and see it is a secure connection. You should also no longer be able to access the web GUI's of those containers by any other means, as the ports are not being exposed, so only Nginx can access them using the domain name.  
 
