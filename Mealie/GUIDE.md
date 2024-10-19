# Mealie Recipe Cookbook

This docker container exposes port 9925 (host) to port 9000 (container) for the web GUI. 
  
Once the container is spun up, navigate to `x.x.x.x:9000` to log in, where `x.x.x.x` is the IP address specified in the configuration file.   

Once Nginx Proxy Manager is configured, go back and remove ports section, as the web GUI will only be accessible through the Proxy. Also, remove the default network created with this container, and instead attach this to the Nginx proxy docker network.  


## Resources:
* Setup
  * [Mealie Wiki -Installation](https://docs.mealie.io/documentation/getting-started/installation/installation-checklist/)  


## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/mealie
  ```

Create a directory to hold persistant storage:  

  ```
  mkdir /opt/mealie/data
  ```  

Change into the `/opt/mealie` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now spin up the container with `docker compose up -d`, and on any device on the home network, navigate to the web GUI on your browser at `[IP-ADDRESS]:9925` to log in.  


## Nginx Proxy:  

After ensuring service can run exposed on ports, move the web GUI behind the Nginx Proxy and remove the ports exposure.  

On Nginx Proxy Manager, create a subdomain for Mealie, such as `mealie.[YOUR-SUBDOMAIN].duckdns.org`, and add the SSL Certificate to it.  

By attaching this docker container to the `npm_proxy` network, and removing the port exposure of 9925, you should now be able to access Mealie at that subdomain through Nginx Proxy Manager, where you will see the browser connection secured and encrypted with SSL.  

Now, you can finish setting up Mealie, create an account, and log in.  
