### Migrating Heimdall to LXC
----------

Migrating one Heimdall instance to another can be a bit challenging. The project devs only provide a simple JSON import/export function, and while it does indeed export all of your entries, it fails to include tags of any kind or any custom icons. 

The only way to migrate is to install the Heimdall LXC container, run it once, shut the container down, and then copy the appropriate files and folders to this container.

### Part 1: Locate the files on the old container.
1. I was using `docker-compose` previously, so all of the pertinent files were saved in a `config` folder. The files and folders you will need to copy are:
   ```
   SupportedApps folder
   avatars folder
   backgrounds folder
   icons folder
   app.sqlite app database
   ```
2. The easiest thing to do is download the whole `config` folder using an SFTP client.

### Part 2: Install Heimdall
1. Follow the directions from here:
   ```
   https://community-scripts.github.io/ProxmoxVE/scripts?id=heimdall-dashboard
   ```
2. Once the new container is running, shut the container down.
3. Connect to the host via SFTP and navigate to the VM storage volume you selected during install and enter the storage folder for the Heimdall container.

   `/vmpool/subvol-100-disk-0` in my case.

4. From there, navigate to `/opt/Heimdall/database` and upload `app.sqlite` overwriting the one that is there.
5. Then navigate to `/opt/Heimdall/storage/app/public` and upload `avatars`, `backgrounds`, and `icons` folders. If there is a folder called `supportedapps` (note case) copy the contents of the `SupportedApps` folder you previously downloaded.
6. Restart the Heimdall container and your account plus all of your bookmarks should now be there.
