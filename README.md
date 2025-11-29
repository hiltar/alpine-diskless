# alpine-diskless
Alpine Linux setup with diskless


# setup-alpine

```
setup-alpine
# Proceed with default options.

# SSH
password # TODO: SSH key auth

# Disk & Install
none
```

# Services


# Filesystem in RAM
```
# For services
echo "tmpfs /mnt/ramdisk tmpfs size=8m,mode=0755 0 0" >> /etc/fstab
# Increase tmp size for lbu
echo "tmpfs /tmp tmpfs defaults,noatime,nosuid,nodev,size=64m 0 0" >> /etc/fstab
```  
`mount -a`

