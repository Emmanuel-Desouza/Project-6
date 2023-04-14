# Web Solution with WordPress

## LAUNCHING AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”

### Step 1 — Prepare a Web Server: Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as the Web Server EC2, each of 10 GiB.

### Attaching all three volumes one by one to the Web Server EC2 instance

### Use 'lsblk' command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

`lsblk`

![viewing block devices](./images/lsblk.png)

### Using df -h command to see all mounts and free space on your server

`df -h`

![mounts](./images/df-h.png)

### Using the 'gdisk' utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdf`

![partitioning](./images/xvdf.png)

![partitioning](./images/xvdf1.png)

`sudo gdisk /dev/xvdg`

![partitioning](./images/xvdg.png)

![partitioning](./images/xvdg1.png)

`sudo gdisk /dev/xvdh`

![partitioning](./images/xvdh.png)

![partitioning](./images/xvdh1.png)

### Using the 'lsblk' utility to view the newly configured partition on each of the 3 disks.

`lsblk`

![viewing partitions](./images/lsblk2.png)

### Install lvm2 package using sudo yum install lvm2. 

`sudo yum install lvm2`

![yum install](./images/yum-install.png)

### Run sudo lvmdiskscan command to check for available partitions.

`sudo lvmdiskscan`

![disk scan](./images/lvmdiskscan.png)

### Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

### Using pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

![creating physical volume](./images/pv-create.png)

### Verifying that the Physical volumess have been created successfully by running `sudo pvs`

![verifying physical volume](./images/sudo-pvs.png)

### Using vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![creating volume group](./images/vg-create.png)
### Verify that your VG has been created successfully by running `sudo vgs`

![verifying vg creation](./images/sudo-vgs.png)

### Using `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

`sudo lvcreate -n apps-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`

![logical volume creation](./images/lv-create.png)

### Verify that your Logical Volume has been created successfully by running `sudo lvs`

![verifying logical volume](./images/sudo-lvs.png)

### Verifying the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![verifying setup](./images/sudo-vgdisplay.png)

`sudo lsblk`

![verifying setup](./images/sudo-lsblk.png)

### Using `mkfs.ext4` to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![formatting LVs](./images/sudo-mkfs.png)

### Creating /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

### Creating /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

### Mount /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

![create and mount](./images/sudos.png)

### Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![rsync utility](./images/sudo-rsync.png)

### Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step above is very important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

![mounting /var/log on lv](./images/mount-var-log.png)

### Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

![restoring log files](./images/rsync-av.png)

### Updating /etc/fstab file so that the mount configuration will persist after restart of the server.

## UPDATE THE `/ETC/FSTAB` FILE

### The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`

![UUID](./images/sudo-blkid.png)

`sudo vi /etc/fstab`

### Updating /etc/fstab in this format using my own UUID, without the leading and ending quotes.

![Updating /etc/fstab](./images/vi-etc.png)

### Test the configuration and reload the daemon

`sudo mount -a`
`sudo systemctl daemon-reload`

![testing config and reloading daemon](./images/mount-systemctl.png)
### Verifying the setup by running `df -h`,

`df -h`

![verifying setup](./images/dfh.png)

## Step 2 — Prepare the Database Server

### Launching a second RedHat EC2 instance that will have a role – ‘DB Server’
### Repeating the same steps as for the Web Server, but instead of apps-lv, we create db-lv and mount it to /db directory instead of /var/www/html/.

## STEPS REPEATED ON THE DB SERVER AS DESCRIBED ABOVE
### Use `lsblk` command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

![inspecting block devices attached to DB server](./images/lsblk-db.png)

### Using `df -h` command to see all mounts and free space on your server

![seeing mounts and free spaces on the DB server](./images/dfh-db.png)

### Use `gdisk` utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdf`
`sudo gdisk /dev/xvdg`
`sudo gdisk /dev/xvdh`

![partition xvdf](./images/xvdf-db.png)
![partition xvdg](./images/xvdg-db.png)
![partition xvdh](./images/xvdh-db.png)

### Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![viewing newly configured partitions](./images/lsblk-db2.png)

### Installing lvm2 package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

`sudo yum install lvm2`

![installing lvm2](./images/lvm2-db.png)

`sudo lvmdiskscan`

![available partitions](./images/lvmdiskscan-db.png)

### Using pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

### Verify that your Physical volume has been created successfully by running `sudo pvs`

![verifying PVs](./images/sudopvs-db.png)

### Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

### Verify that your VG has been created successfully by running `sudo vgs`

![verifying PVs](./images/vgcreate-db.png)

### Using lvcreate utility to create 2 logical volumes. db-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: db-lv will be used to store data for the database while, logs-lv will be used to store data for logs.

`sudo lvcreate -n db-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`

### Verify that your Logical Volume has been created successfully by running `sudo lvs`

![creating LVs](./images/lvcreate-db.png)

### Verifying the entire setup:

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`


![VG, PV, LV display](./images/vgdisplay-db.png)

`sudo lsblk`

![lsblk](./images/lsblk-db3.png)