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
   pct set container_id -mp0 /path/to/photos,mp=/photos
   ```
   where `container_id` is the number listed in front of the container on the dashboard
   and `/path/to/photos` is the location of the media files on the host (in my case it is an NFS share mounted on `/mnt/pve/photos`)
4. Start the container.
5. Log in to the application as an administrator.
6. Enter the admin menu and select `External Libraries`.
7. Create an external library that points to the network share mounted inside the container as noted in step 3.
8. Use that external library as the storage for all of your albums.
