# Vaultwarden Password Manager

This docker container exposes port 8080 (host) to port 80 (container) for the web GUI. 
  
Once the container is spun up, navigate to `x.x.x.x:8080` to log in, where `x.x.x.x` is the IP address specified in the configuration file.   

Later, once Nginx Proxy Manager is configured, go back and remove ports section, as the web GUI will only be accessible through the Proxy. Also, you will remove the default network created with this container, and instead attach this to the Nginx proxy docker network.  


## Resources:
* Setting Up Vaultwarden
  * [Vaultwarden Wiki - Using Docker Compose](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose)
    
* Backups
  * [Vaultwarden Wiki - Backing Up Your Vault](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault)  


## Duck DNS Setup:

Vaultwarden requires secure SSL certificates on the web browser GUI. As such, you will need a domain name to link to a certificate. This setup will use Duck DNS again, this time pointing to an internal IP address.    

Go to [duckdns.org](https://www.duckdns.org/) and sign in using your preferred method. Then, create a domain with any name that you like, as long as it isn’t already taken.  

Replace the IP address on this subdomain with the address of your host server on the home network, so that this domain can only be accessed internally within your home network, i.e. `192.168.10.x`.     


## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/vaultwarden
  ```

Create a directory to hold persistant storage:  

  ```
  mkdir /opt/vaultwarden/data
  ```  

Change into the `/opt/vaultwarden` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now spin up the container with `docker compose up -d`, and on any device on the home network, navigate to the web GUI on your browser at `[IP-ADDRESS]:8080` to log in.  

You should get an error, that your web browser is unsecured.  Follow the steps for [setting up Nginx Proxy Manager](/Nginx-Proxy-Manager/GUIDE.md), then return back here to finish setting up Vaultwarden.  


## SSL Encryption:  

After setting up Nginx Proxy Manager, you should have created a subdomain for Vaultwarden, such as `vw.[YOUR-SUBDOMAIN].duckdns.org`. By attaching this docker container to the `npm_proxy` network, and removing the port exposure of 8080, you should now be able to access Vaultwarden at that subdomain through Nginx Proxy Manager, where you will see the browser connection secured and encrypted with SSL.  

Now, you can finish setting up Vaultwarden, create an account, and log in.  

Download the Bitwarden mobile app, desktop app, and browser extension, and link your locally-hosted Vaultwarden service to it with the domain address created in Nginx Proxy Manager.  

You should now have a successful password manager service running, with no exposed ports and hosted completely internally within the home network.  
