## Update the configured APT repositories
First, make sure that the system is using the latest Proxmox VE 8.4 packages:

```
apt update
apt dist-upgrade
pveversion

```
The last command should report at least 8.4.1 or newer.

Note: For hyperconverged Ceph setups, ensure that you are running Ceph Squid (version 19). Check the output of ceph --version to be sure. If you are not running Ceph Squid, see the upgrade guide for Ceph Reef to Squid and complete the upgrade first. Do not proceed with any of the steps below before upgrading to Ceph Squid.

## Update Debian Base Repositories to Trixie
### Update all Debian and Proxmox VE repository entries to Trixie.

```
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/pve-enterprise.list

```
Ensure that there are no remaining Debian Bookworm specific repositories left, otherwise you can put a # symbol at the start of the respective line to comment these repositories out. Check all entries in the /etc/apt/sources.list and /etc/apt/sources.list.d/pve-enterprise.list, for the correct Proxmox VE 9 / Debian Trixie repositories see Package Repositories.

### Add the Proxmox VE 9 Package Repository
If you are using the enterprise repository, you can add the Proxmox VE 9 enterprise repository in the new deb822 style. Run the following command to create the related pve-enterprise.sources file:

```
cat > /etc/apt/sources.list.d/pve-enterprise.sources << EOF
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF

```
After you added the new enterprise repository as above, check that apt picks it up correctly. You can do so by first running apt update followed by apt policy. Make sure that no errors are shown and that apt policy only outputs the desired repositories. Then you can remove the old /etc/apt/sources.list.d/pve-enterprise.list file. Run apt update and apt policy again to be certain that the old repo has been removed.

If using the no-subscription repository, see Package Repositories. You should be able to add the Proxmox VE 9 no-subscription repository with this command:

```
cat > /etc/apt/sources.list.d/proxmox.sources << EOF
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF

```
As with the enterprise repository, make sure that apt picks it up correctly with apt update followed by apt policy. Then remove the previous Proxmox VE 8 no-subscription repository from either the /etc/apt/sources.list, /etc/apt/sources-list.d/pve-install-repo.list or any other .list file you may have added it to. Run apt update and apt policy again to be certain that the old repo has been removed.

### Update the Ceph Package Repository
Note: For hyper-converged ceph setups only, check the ceph panel and configured repositories in the Web UI of this node, if unsure.

Replace any ceph.com repositories with proxmox.com ceph repositories.

Note: At this point a hyper-converged Ceph cluster installed directly in Proxmox VE must run Ceph 19.2 Squid, otherwise you need to upgrade Ceph first before upgrading to Proxmox VE 9 on Debian 13 Trixie! You can check the current Ceph version in the Ceph panel of each node in the Web UI of Proxmox VE.

Since Proxmox VE 8 there also exists an enterprise repository for ceph, providing the best choice for production setups. Execute the command below to add the Trixie-based Ceph enterprise repository in the new deb822 style:
```

cat > /etc/apt/sources.list.d/ceph.sources << EOF
Types: deb
URIs: https://enterprise.proxmox.com/debian/ceph-squid
Suites: trixie
Components: enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

Make sure that apt picks it up correctly by running apt update first and then apt policy. There should be no errors and the new repository should show up correctly in the output of apt policy. Then you can remove the old /etc/apt/sources.list.d/ceph.list file. You can run apt update and then apt policy again to make sure it has been properly removed.

If updating fails with a 401 error, you might need to refresh the subscription first to ensure new access to ceph is granted, do this via the Web UI or pvesubscription update --force.

If you do not have any subscription you can use the no-subscription repository, add it with the following command in the new deb822 style:
```

cat > /etc/apt/sources.list.d/ceph.sources << EOF
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF

```
As with the enterprise repository, make sure that apt picks it up correctly with apt update followed by apt policy. Then you can remove the old /etc/apt/sources.list.d/ceph.list file.

If there is a backports line, remove it - the upgrade has not been tested with packages from the backports repository installed.

### Refresh Package Index
Update the repositories' package index and verify that no error is reported:

`apt update`

### Upgrade the system to Debian Trixie and Proxmox VE 9.0
Note that the time required for finishing this step heavily depends on the system's performance, especially the root filesystem's IOPS and bandwidth. A slow spinner can take up to 60 minutes or more, while for a high-performance server with SSD storage, the dist-upgrade can be finished in under 5 minutes.

Start with this step, to get the initial set of upgraded packages:
```

apt dist-upgrade

```
During the above step, you will be asked to approve changes to configuration files and some service restarts, where the default config has been updated by their respective package.

You may also be shown the output of apt-listchanges, you can simply exit there by pressing "q". If you get prompted for your default keyboard selection, simply use the arrow keys to navigate to the one applicable in your case and hit enter.

For questions about service restarts (like Restart services during package upgrades without asking?) use the default if unsure, as the reboot after the upgrade will restart all services cleanly anyway.

It's suggested to check the difference for each file in question and choose the answer accordingly to what's most appropriate for your setup.

Common configuration files with changes, and the recommended choices are:

/etc/issue -> Proxmox VE will auto-generate this file on boot, and it has only cosmetic effects on the login console.
Using the default "No" (keep your currently-installed version) is safe here.
/etc/lvm/lvm.conf -> Changes relevant for Proxmox VE will be updated, and a newer config version might be useful.
If you did not make extra changes yourself and are unsure it's suggested to choose "Yes" (install the package maintainer's version) here.
/etc/ssh/sshd_config -> If you have not changed this file manually, the only differences should be a replacement of ChallengeResponseAuthentication no with KbdInteractiveAuthentication no and some irrelevant changes in comments (lines starting with #).
If this is the case, both options are safe, though we would recommend installing the package maintainer's version in order to move away from the deprecated ChallengeResponseAuthentication option. If there are other changes, we suggest to inspect them closely and decide accordingly.
/etc/default/grub -> Here you may want to take special care, as this is normally only asked for if you changed it manually, e.g., for adding some kernel command line option.
It's recommended to check the difference for any relevant change, note that changes in comments (lines starting with #) are not relevant.
If unsure, we suggested to selected "No" (keep your currently-installed version)
/etc/chrony/chrony.conf -> If you made local changes you might want to move them out of the global config into the conf.d or, for custom time sources, the sources.d folder.
See the /etc/chrony/conf.d/README and /etc/chrony/sources.d/README files on your system for detaily.
If you did not make extra changes yourself and are unsure it's suggested to choose "Yes" (install the package maintainer's version) here.
Check Result & Reboot Into Updated Kernel
If the dist-upgrade command exits successfully, you can re-check the pve8to9 checker script and reboot the system in order to use the new Proxmox VE kernel.

Please note that you should reboot even if you already used the 6.14 kernel previously, through the opt-in package on Proxmox VE 8. This is required to guarantee the best compatibility with the rest of the system, as the updated kernel was (re-)build with the newer Proxmox VE 9 compiler and ABI versions.
