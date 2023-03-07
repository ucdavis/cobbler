## Build
`./docker/debs/build-and-install-debs.sh deb11 docker/debs/Debian_11/Debian11.dockerfile`

deb built in: `deb-build/DEBS/all/cobbler_3.4.0-1debian11_all.deb`

## Install
`apt install python3-ldap python3-dnspython`

## Config files to check:
```
/etc/cobbler/dhcp.template
/etc/cobbler/named.template
/etc/cobbler/settings.yaml
/etc/apache2/conf-available/cobbler.conf
```
## Post installation bootstrapping

```bash
cd /var/lib/cobbler/collections/images/
wget https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-live-server-amd64.iso
mount -o loop,ro ubuntu-22.04.2-live-server-amd64.iso /mnt/

cobbler import --path=/mnt/ --name=ubuntu-22.04

umount /mnt

cobbler profile remove --name=ubuntu-22.04-casper-x86_64

cobbler distro edit --name=ubuntu-22.04-casper-x86_64 --kernel-options="swapaccount=1 transparent_hugepage=madvise net.ifnames=0 biosdevname=0" --kernel-options-post="swapaccount=1 transparent_hugepage=madvise net.ifnames=0 biosdevname=0"

cobbler profile add --name=compute-22.04  --distro=ubuntu-22.04-casper-x86_64 --filename="/pxelinux.0"
cobbler profile add --name=infrastructure --distro=ubuntu-22.04-casper-x86_64

cobbler list
```
