# alpine-diskless
Alpine Linux setup with diskless mode for Raspberry Pi


# setup-alpine

```
setup-alpine
# Proceed with desired options.

# SSH
password # TODO: SSH key auth

# Disk & Install
none
```

# Configurations
`cmdline.txt`  
```
modules=loop,squashfs,sd-mod,usb-storage quiet console=tty1
```  
`usercfg.txt`  
```
gpu_mem=8
dtparam=audio=off
dtoverlay=disable-bt
hdmi_enable=0
```  

# Services

`vi /etc/init.d/service`  
```
#!/sbin/openrc-run

name="service"
description="Run service from ramdisk"
command="/mnt/ramdisk/service/service"
command_background=true
pidfile="/run/${RC_SVCNAME}.pid"
    
start_pre() {
    if [ ! -f /mnt/ramdisk/service/service ]; then
        mkdir /mnt/ramdisk/service
        cp /media/mmcblk0p1/service/service /mnt/ramdisk/service || return 1
    fi
}

start() {
    ebegin "Starting ${name}"
    start-stop-daemon --start --exec "${command}" \
        --pidfile "${pidfile}" \
        --background \
        --make-pidfile \
        --stdout /var/log/service.log \
    eend $?
}
```  

## Enable service
`chmod +x /etc/init.d/service`  
`rc-update add service default`  
`mkdir /opt/service/ # If service saves configuration files or data`  

# LBU
Local backup utility(lbu) is the Alpine Linux tool to manage Diskless Mode installations. For these installations, `lbu` tool must be used to commit the changes whenever Alpine Package Keeper is used.

When Alpine Linux boots in diskless mode, it initially only loads a few required packages from the boot device. However, local adjustments to what-gets-loaded-into-RAM are possible, e.g. installing a package or adjusting the configuration files in /etc. The modifications can be saved with `lbu` tool to an overlay file i.e apkovl file that can be automatically loaded when booting, to restore the saved state.

By default, an `lbu commit` only stores modifications below /etc, with the exception of the /etc/init.d/ directory. If a user was created during the setup-alpine script, that user's home directory is also added to the paths that lbu will backup up. However, `lbu add` enables modifying that set of included files, and can be used to specify additional files or folders. 

```
lbu add /opt/service/
lbu add /etc/init.d/service
lbu add /etc/wpa_supplicant/ # If using WiFi network
lbu commit
```  

# Filesystem in RAM
```
# For services - This example uses 8MB RAM
echo "tmpfs /mnt/ramdisk tmpfs size=8m,mode=0755 0 0" >> /etc/fstab
# Increase tmp size for lbu - 64MB RAM
echo "tmpfs /tmp tmpfs defaults,noatime,nosuid,nodev,size=64m 0 0" >> /etc/fstab
```  
`mount -a`


# Crontab
`crontab -e`  
```
# min hour day month weekday command
  1   1    *   *     7       lbu status | grep -q . && /usr/sbin/lbu commit # Backup system every week at Sunday              
  2   4    *   1/3   *       apk update && apk upgrade -a --no-interactive  # Upgrade packages every Q
  3   4    *   1/3   *       rc-service sshd restart
```
