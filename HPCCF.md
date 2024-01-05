## Build
https://github.com/ucdavis/cobbler
* `Sync fork`

`git pull`
`apt install podman`
`./docker/debs/build-and-install-debs.sh --with-podman deb11 docker/debs/Debian_11/Debian11.dockerfile`

deb built as: `deb-build/cobbler_3.4.0_all.deb`

Prune the docker containers:
* `sudo podman container prune`
* `sudo podman image prune -a`

## Install
```
apt install python3-ldap python3-dnspython gunicorn pxelinux ipxe shim-signed apache2 tftpd-hpa fence-agents xorriso python3-cheetah python3-netaddr python3-pymongo python3-schema bind9 isc-dhcp-server
dpkg -i deb-build/cobbler_3.4.0_all.deb

# Optional, purge podman
apt remove --purge podman
apt autoremove --purge
```

## Config files to modify:
```
/etc/cobbler/dhcp.template
/etc/cobbler/named.template
/etc/cobbler/secondary.template
/etc/cobbler/settings.yaml
/etc/apache2/sites-available/cobbler.conf
```

## Post installation bootstrapping
```bash
# cp, NOT symlink. tftpd will not follow symlinks
cp -a  /usr/lib/PXELINUX/pxelinux.0 /srv/tftp/
cp -a /usr/lib/syslinux/modules/bios/ldlinux.c32 /srv/tftp/

a2enconf cobbler.conf
a2enmod proxy_http
systemctl restart apache2
systemctl restart tftpd-hpa.service

export LTS=22.04.3

mkdir -p /var/www/cobbler /var/www/cobbler-html/ISOs/
cd /var/www/cobbler-html/ISOs/

wget -O ubuntu-22.04-casper-x86_64.iso https://releases.ubuntu.com/jammy/ubuntu-$LTS-live-server-amd64.iso

mkdir /var/lib/bind/data
chown bind: /var/lib/bind/data

cd /etc/systemd/system
ln -s /etc/cobbler/cobblerd.service
ln -s /etc/cobbler/cobblerd-gunicorn.service
systemctl daemon-reload
systemctl enable --now cobblerd.service cobblerd-gunicorn.service

```

## Import Ubuntu Jammy as a distro and create the profile
```bash
# VERY IMPORTANT that this finish successfully
cobbler sync

ln -s /var/www/cobbler-html/ISOs/ubuntu-22.04-casper-x86_64.iso /var/lib/cobbler/collections/images/

#cd /var/lib/cobbler/collections/images/
#wget https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-live-server-amd64.iso
mount -o loop,ro /var/lib/cobbler/collections/images/ubuntu-22.04-casper-x86_64.iso /mnt/

cobbler import --path=/mnt/ --name=ubuntu-22.04
# cobbler import also creates a matching profile which we don't want.
cobbler profile remove --name=ubuntu-22.04-casper-x86_64

umount /mnt

cobbler distro edit --name=ubuntu-22.04-casper-x86_64 --kernel-options="swapaccount=1 transparent_hugepage=madvise net.ifnames=0 biosdevname=0 ipv6.disable=1" --kernel-options-post="swapaccount=1 transparent_hugepage=madvise net.ifnames=0 biosdevname=0 ipv6.disable=1"

cobbler profile add --name=compute-22.04  --distro=ubuntu-22.04-casper-x86_64 --filename="/pxelinux.0"
cobbler profile add --name=infrastructure --distro=ubuntu-22.04-casper-x86_64

cobbler list
```

## Install cheeto
```bash
# /var/www/cobbler-html/ will populatd from puppet

cd /root/
ln -s /opt/hpccf/sbin cobbler
```
