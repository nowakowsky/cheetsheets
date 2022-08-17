# Encrypted transfer of disk using socat and dd
I've used this method to migrate vps but there are many other usecases.

#### 1\. Server (source):

```bash
openssl req -newkey rsa:2048 -nodes -keyout encrypted.key -x509 -days 10 -subj '/CN=example.com/O=acme/C=US' -out encrypted.crt
```

```bash
cat encrypted.key encrypted.crt > encrypted.pem
```

```bash
# change sdb to your device!
sudo dd if=/dev/sdb bs=2M | socat STDIO OPENSSL-LISTEN:9876,cert=encrypted.pem,verify=0,reuseaddr
```

#### 2\. Client (destination):

```bash
# change ip to your server
socat -u OPENSSL:IP.IP.IP.IP:9876,verify=0 OPEN:sda.img,creat
```

#### 3\. Restore

```bash
sudo dd if=/tmp/sdb.img of=/dev/sda bs=2M status=progress
```

#### 4\. Fix grub - credits: [OVH Support](https://docs.ovh.com/us/en/public-cloud/repairing-the-grub-bootloader/)

```bash
mount /dev/sdb1 /mnt
mount -o bind /proc /mnt/proc
mount -o bind /sys /mnt/sys
mount -o bind /dev /mnt/dev
chroot /mnt /bin/bash
```

```bash
# grub
grub-install /dev/sdb
update-grub
```

```bash
# grub2  
grub2-install /dev/sdb
grub2-mkconfig -o /boot/grub2/grub.cfg
```
