# Uptime Kuma Health Monitor

This docker container exposes port 3010 (host) to port 3001 (container) for the web GUI. 
  
Once the container is spun up, navigate to `x.x.x.x:3010` to log in, where `x.x.x.x` is the IP address of your host.   

Later, once Nginx Proxy Manager is configured, go back and remove ports section, as the web GUI will only be accessible through the Proxy. Also, you will remove the default network created with this container, and instead attach this to the Nginx proxy docker network.  

## Resources:
* [Jim's Garage - Uptime Kuma](https://www.youtube.com/watch?v=0FId6vahLAI&t=979s)
* [Setting up Uptime Kuma with Docker Compose](https://docs.techdox.nz/uptimekuma/)
* [How to Monitor Docker Containers with Socket](https://github.com/louislam/uptime-kuma/wiki/How-to-Monitor-Docker-Containers/d37b9fd9c9e106b21e28af9644307ca1902ea341)


## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/uptimekuma
  ```

Create a directory to hold persistant storage:  

  ```
  mkdir /opt/uptimekuma/data
  ```  

Change into the `/opt/uptimekuma` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now spin up the container with `docker compose up -d`, and on any device on the home network, navigate to the web GUI on your browser at `[IP-ADDRESS]:3010` to log in.  

Ensure everything is working properly, then go and set up a proxy host on Nginx Proxy Manager for the Web UI. Remove the ports section from the docker-compose.yml as you will no longer expose ports to the host for the Web UI.  
