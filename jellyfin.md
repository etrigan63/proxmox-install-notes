## Jellyfin Server + Nvidia GPU Passthrough

### Part 1: Nvidia driver install on host
----------
1. Download Nvidia Linux drivers from Nvidia website.
2. Upload drivers to Proxmox host via SFTP
3. Make sure `gcc`, `make` and Linux kernel header files are installed.
   ```
   apt install gcc make
   apt install  pve-headers-`uname -r`
4. Make the Nvidia driver package executable

   `chmod +x NVIDIA-Linux-x86_64-version.number.run`

   (change `version.number` to match whatever you downloaded)
5. Run the installer.

   `./NVIDIA-Linux-x86_64-version.number.run`
   The installer will blacklist the `nouveau` driver for you.
6. Reboot the server.
7. Run `nvidia-smi` to make sure the host sees the GPU.
8. Edit `modules.conf`:

   `nano /etc/modules-load.d/modules.conf`
9. Add in:
```
   nvidia
   nvidia-modeset
   nvidia_uvm
```
10. Save the file and regenerate the kernel intramfs:

   `update-intramfs -u`
11. Create the udev rules:

   `nano /etc/udev/rules.d/70-nvidia.rules`
   Add these lines:
```
   KERNEL=="nvidia", RUN+="/bin/bash -c '/usr/bin/nvidia-smi -L && /bin/chmod 666 /dev/nvidia*'"
   KERNEL=="nvidia_modeset", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -m && /bin/chmod 666 /dev/nvidia-modeset*'"
   KERNEL=="nvidia_uvm", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -u && /bin/chmod 666 /dev/nvidia-uvm*'
```
13. Save and reboot the server.
14. Run `nvidia-smi` to double check.

### Part 2: Install Jellyfin
----------
1. Install Jellyfin container via the helper script. Open the host console and type:
   `bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/jellyfin.sh)"`
   This will install a fully configured Jellyfin server. We just need to make a couple of modifications.
2. Shutdown the container.
3. Open the host console and type the following:
   `pct set container_id -mp0 /path/to/media,mp=/media`
   where `container_id` is the number listed in front of the container on the dashboard
   and `/path/to/media` is the location of the media files on the host (in my case it is an NFS share mounted on `/mnt/pve/media`)
4. Start the container.

### Part 3: Add GPU Passthrough
----------
1. Go to the host console and enter:
   `ls -l /dev/nv*`
   This will list Nvidia devices recognized by the host (and any NVMe drives as well) and will contain the devices and hardware ID's that will be needed for the passthrough.
2. SSH into the host so that you can have the hardware listing side-by-side.
3. Edit the container `.conf` file:
   `nano /etc/pve/lxc/<container id>.conf`
4. Add the following lines:
   `#cgroup access
   lxc.cgroup2.devices.allow: c 195:0 rw
   lxc.cgroup2.devices.allow: c 195:255 rw
   lxc.cgroup2.devices.allow: c 195:254 rw
   lxc.cgroup2.devices.allow: c 510:0 rw
   lxc.cgroup2.devices.allow: c 510:1 rw
   lxc.cgroup2.devices.allow: c 10:144 rw

   #device files   
   lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
   lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
   lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
   lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
   lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
   lxc.mount.entry: /dev/nvram dev/nvram none bind,optional,create=file`
   Please note that `nvidia-modeset` may not be listed on the other session. If that is the case, comment out the third line of the `cgroup` and `device` sections. Modify the number of the `cgroup` section to match the listing in the other session.
5. Copy the Nvidia driver installer to the container's `/root` folder. This is located on the disk you specified during the container install. In my case it is:
   `/vmpool/subvol-101-disk-0/root`
6. Log in to the container's console.   
7. Make the installer executable.
   `chmod +x NVIDIA-Linux-x86_64-version.run`
8. Run the installer with `--no-kernel-modules`
   `./NVIDIA-Linux-x86_64-version.run --no-kernel-modules`
9. Run `nvidia-smi` inside the container to make sure it can see the GPU.

### Part 4: Configure Jellyfin
----------
1. Login to the Jellyfin server and follow the setup guide.
2. You should be able to mount your media libraries.
3. Once that is complete, go to the Admin dashboard and enable hardware transcoding. Select Nvidia NVENC and hit save. Test video playback.
