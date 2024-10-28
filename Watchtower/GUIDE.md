# Watchtower Docker Container Automated Updates

This docker container exposes no ports and has no web GUI.  
  
## Resources:
* [Jim's Garage - Watchtower](https://www.youtube.com/watch?v=mSSlrRgSAP4)
* [Watchtower Wiki](https://containrrr.dev/watchtower/)


## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/watchtower
  ```  

Change into the `/opt/watchtower` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, and save it.  

Now spin up the container with `docker compose up -d`, and on any device on the home network.  

Ensure everything is working properly by checking Watchtower's docker container logs either through CLI or on Portainer, and seeing how other docker containers are updated. 
