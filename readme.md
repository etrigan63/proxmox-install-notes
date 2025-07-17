## Proxmox Install Notes

Notes on how I set up the LXC containers on my Proxmox server.

So far, every one except `nextcloud` is based on community helper scripts. The `nextcloud` container is based on an Ubuntu 24.04 LTS container with the `nextcloud snap` providing the server. The snap is self-upgrading and Ubuntu has an established upgrade methodology.
