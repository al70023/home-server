# Restic Docker Container Backups

This docker container exposes no ports and has no web GUI.  

## Resources:
* [Jim's Garage - Restic Backups](https://www.youtube.com/watch?v=WBBTC5WfGis)


## Docker Setup:  

Create a new directory for the service:

  ```
  mkdir /opt/restic
  ```  

Change into the `/opt/restic` directory, and create a new docker compose file for the service:

  ```
  sudo touch docker-compose.yml
  sudo nano docker-compose.yml 
  ```

Use [the attached configuration file](docker-compose.yml), paste it into the `docker-compose.yml`, make necessary changes, and save it.  

Now spin up the container with `docker compose up -d`, and on any device on the home network.  

To ensure proper setup, check the restic docker container logs via Portainer. It should look something like this:  

![image](https://github.com/user-attachments/assets/0de519bf-8ef3-497b-8539-c41a106eb61a)  

You may now navigate to the directory specified in your docker compose file for the Restic backups (i.e. `/home/backups`) and check that there is data int the subdirectories.  


## Restoring from Backups:  

To restore from your Restic backups, you will send all the docker data into your created `tmp-for-restore` subdirectory within the `home/backups` directory you created.  

You may first check that this `tmp-for-restore` folder is empty, to see how it will change after the restore.  

On your host, connect to the `restic` docker container and check your snapshots via Portainer, or through the command line with the following command:

  ```
  docker exec -it restic restic snapshots -H [YOU_SPECIFIED_HOSTNAME]
  ```

Now, copy the snapshot ID, and use that in the following command to create a restore:  

  ```
  docker exec -it restic restic restore --target /tmp-for-restore [COPIED_SNAPSHOT_ID]
  ```

Once that completes, you should be able to navigate to your temp restore location at `/home/backups/tmp-for-restore`, and see all your docker containers saved.  

Now, you can safely shut down your docker container to restore, copy over all of the data and files from your backup restore, spin the container back up, and you should have successfully restored that container.
