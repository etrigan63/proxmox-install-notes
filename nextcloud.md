### Nextcloud LXC Container
----------

### TrueNAS Setup - Datasets and NFS Share 

1. Create a dataset.
2. Set owner/group to `root/root`.
3. Set ACL to `POSIX_HOME`. This sets permissions to 770 which is required for Nextcloud.
4. Create NFS share on the Shares panel. Make sure to add the local network in CIDR notation. Then click on Advanced Options and set the Mapall User and Mapall Group each to `root`. This allows the share to be accessed as root from the container.

### Proxmox Host Setup

1. Use the Proxmox web interface to mount the NFS share locally.
2. `chown -R 100000:100000 /path/to/nextcloud` - This sets the owner of the mount to immich:root inside the container.
3. `chmod -R 770 /path/to/nextcloud` - To make sure file permissions are correct.


### Nextcloud LXC Container Setup

1. If you have not already done so, download the Ubuntu 24.04 LTS container template. This will be used to create the container.
2. Click the `Create CT` button.
3. Create a container with the following parameters:
   ```
   Hostname: nextcloud
   Password: enter the root password
   Template: Ubuntu 24.04
   Local Disk Space: 128GB (just to have maneuvering room for the database)
   CPU: 2 
   RAM: 2048
   Network: Static IP - up to you
   DNS: use defaults
4. Build the container. (Note: the container is not set to autostart at boot time. That will be addressed later.)
5. Once the container is built, click on it once to select it and click on the `Options` panel.
6. Click on `Features` and then click the `Edit` button.
7. Enable `FUSE` and then click OK.
8. Start the container and enter its console using the `root` account and the password you set for it in step 3 of this section.
9. Enter `mkdir /media/folder-name` where folder-name is the one you used on the host (cuts down on the confusion). Check to make sure it is owned by `root`.
10. Enter `apt update && apt upgrade -y` to update the container OS.
11. Enter `apt install snapd` to install the Snap service.
12. Enter `snap install nextcloud`. If it errors, do it again. This is fairly normal.
13. Shutdown the container.
14. Open the host console.
15. Type the following:
   ```
   pct set container_id -mp0 /path/to/nextcloud,mp=/media/nextcloud
   ```
   where `container_id` is the number listed in front of the container on the dashboard
   and `/path/to/nextcloud` is the location of the media files on the host (in my case it is an NFS share mounted on `/mnt/pve/nextcloud`)

16. Restart the container.
17. Open the container console and check that the bind is successful. Test that the `root` account owns the share and the permissions are 770. Assuming this works, we are ready to configure the snap.
18. Enter `snap connect nextcloud:removable-media` to grant the snap access to the `/media` folder.
19. Enter `nextcloud.manual-install username password` to setup you base install with and add an admin account. This will take a few minutes.
20. Enter `snap stop nextcloud` to stop the Nextcloud server.
21. Enter `mv /var/snap/nextcloud/common/nextcloud/data /media/folder-name`.
22. Enter `nano /var/snap/nextcloud/current/nextcloud/config/config.php` and change the `'datadirectory' => ` from the default to `/media/folder-name/data`. Then edit `'trusted_domains'` and add `1 => 'container-ip-address',` just below the `loclhost` entry. Replace `container-ip-address` with the IP address you selected in step 3 of the section.
23. Save the file.
24. Enter `snap start nextcloud`.
25. If all goes well, you should be greeted with the Nextcloud login prompt. Use the credentials you entered in step 19 of this section to login. If this works, you are good to go.

### Nextcloud Migration

1. Nextcloud now has a User Migration Tool in the App Store. Doesn't work as smoothly as they promise. There is a bug in that the zip library used requires that the export bundle be manually rebuilt. See below.
2. Upload limits need to be modified. Use the following:
   ```
   snap set nextcloud php.upload-max-filesize=100G
   snap set nextcloud php.post-max-size=100G
   snap set nextcloud php.max-input-time=3600
   snap set nextcloud php.max-execution-time=3600

3. Enter `snap restart nextcloud` to enable.
4. Export user data using the User Migration app. This file can get big. The settings above were for a 93GB export file. YMMV. Also, depending on the size of your export file, you will have to increase the size of the space allocated to the container via the Resources panel on the host. Make it quadruple the size of the export file just to be safe.
5. Download the export file.
6. Rebuild the export file as follows:
   ```
   mkdir extract && cd extract/
   unzip /mnt/data/nextcloud/user.nextcloud_export
   rm /mnt/data/nextcloud/user.nextcloud_export
   zip /mnt/data/nextcloud/user.nextcloud_export -r *
   file /mnt/data/nextcloud/user.nextcloud_export 
   /mnt/data/nextcloud/user.nextcloud_export: Zip archive data, at least v1.0 to extract   
  These commands apply to Linux & Mac. Adjust for actual file location.

7. Upload the file to the new server and run data import. It will take a while, but the upload will complete.

### Proxmox backup notes.

1. Due to the use of snaps inside this container, the only way to back it up is to shut it down during the backup. This will kill the uptime since I don't have a cluster. Luckily, I only backup containers once a month.
