## Build
`./docker/debs/build-and-install-debs.sh deb11 docker/debs/Debian_11/Debian11.dockerfile`

deb built as: `deb-build/cobbler-3.4.0.tar.gz`

## Install
`apt install python3-ldap python3-dnspython gunicorn`

## Config files to check:
```
/etc/cobbler/dhcp.template
/etc/cobbler/named.template
/etc/cobbler/settings.yaml
/etc/apache2/sites-available/cobbler.conf
```
## Post installation bootstrapping

```bash
a2ensite cobbler.conf
a2enmod proxy_http
systemctl restart apache2

mkdir /var/www/cobbler /var/www/cobbler-html
mkdir /var/lib/bind/data
chown bind: /var/lib/bind/data

cd /etc/systemd/system
ln -s /etc/cobbler/cobblerd.service
ln -s /etc/cobbler/cobblerd-gunicorn.service
systemctl daemon-reload
systemctl enable --now cobblerd.service cobblerd-gunicorn.service
```

```bash
# VERY IMPORTANT that this finish successfully
cobbler sync

cd /var/lib/cobbler/collections/images/
wget https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-live-server-amd64.iso
mount -o loop,ro ubuntu-22.04.2-live-server-amd64.iso /mnt/

cobbler import --path=/mnt/ --name=ubuntu-22.04
# cobbler import also creates a matching profile which we don't want.
cobbler profile remove --name=ubuntu-22.04-casper-x86_64

umount /mnt

cobbler distro edit --name=ubuntu-22.04-casper-x86_64 --kernel-options="swapaccount=1 transparent_hugepage=madvise net.ifnames=0 biosdevname=0 ipv6.disable=1" --kernel-options-post="swapaccount=1 transparent_hugepage=madvise net.ifnames=0 biosdevname=0 ipv6.disable=1"

cobbler profile add --name=compute-22.04  --distro=ubuntu-22.04-casper-x86_64 --filename="/pxelinux.0"
cobbler profile add --name=infrastructure --distro=ubuntu-22.04-casper-x86_64

cobbler list
```
