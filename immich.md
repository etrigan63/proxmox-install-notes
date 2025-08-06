### IMMICH CONTAINER SETUP 
----------

1. Install Immich container via the helper script. Open the host console and type:
   ```
   bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/immich.sh)"
   ```
   This will install a fully configured Immich server. We just need to make a couple of modifications.
2. Shutdown the container.
3. Open the host console and type the following:
   ```
   pct set container_id -mp0 /path/to/photos,mp=/mnt/immich
   ```
   where `container_id` is the number listed in front of the container on the dashboard
   and `/path/to/photos` is the location of the media files on the host (in my case it is an NFS share mounted on `/mnt/pve/photos`)
4. `chown -R 100999:100000 /path/to/photos` - This sets the owner of the mount to immich:root inside the container.
5. `chmod -R 770 /path/to/photos` - To make sure file permissions are correct.
6. Start the container.
7. Log in to the application as an administrator.
8. Enter the container console as `root`.
9. Stop immich with `systemctl stop immich-web immich-ml`
10. Edit `/opt/immich/env` and change `IMMICH_MEDIA_LOCATION` to point the container mountpoint (`/mnt/immich` in my case) and save the file.
11. Copy the contents of `/opt/immich/upload` to the new location using `cp -a /opt/immich/upload/* /mnt/immich` (or whatever mointpoint you specified in step 3).
12. Start the `immich` services with `systemctl start immich-web immich-ml`. The app will automatically update any symbolic links to the new path specified in`/opt/immich/.env`.
13. Refresh the `immich` web interface and it should report a lot more free space available on in the lower left hand corner.

