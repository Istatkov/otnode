# Converting docker node to dockerless

## Benefits: 

* Dockerless nodes require less resources and are aimed at advanced users who would like to min/max the efficiency of their nodes in terms of resources.
* You would be saving around 4GB space
* Ability to use one database across multiple nodes on one VPS \(still work in progress\), which should potentially save more space

## Disadvantages:

* Updates require some additional work as of now
* the log is less friendly to read
* autorestarts require additional tools \(i.e. Ansible\)

To perform the transformation, you need to utilize the scripts you can find in [Calvin's Github](https://github.com/calr0x/OT-DockSucker). Always consult those first and the instructions on his Github as those would always be the most up-to-date ones. Should you have any questions, feel free to join the [Advanced Node runners telegram group](https://t.me/otnodes).

There are two ways to perform this process - using Smoothbrain backup, or using local backup.

## Local backup conversion

### Creating the backup

Easiest way is to attach a new Volume to the existing server, create the backup there, detach the volume and attach to the new server. Here are the steps to do it:

Create volume

On the existing server - get the id

```text
df -h
```

![](../.gitbook/assets/image%20%2838%29.png)

Copy on a notepad the number above as we will need it shortly.

On Hetzner and Digital Ocean you can add additional space to your VPS by adding a Volume space. I will walkthrough the process on Hetzner, it is the same on Digital Ocean too.

* Login to your Hetzner Console at [https://console.hetzner.cloud/](https://console.hetzner.cloud/)
* Select your **Project \(default is called Default\)**
* Click on the server. This will bring up server settings.
* Select **Volumes:**

![](../.gitbook/assets/image%20%2813%29.png)

* Click on **Create Volume** button, put the name as vol1 and select manual. For size, select whatever is the size of the VPS's SSD at the moment. 

![](../.gitbook/assets/image%20%2841%29.png)

* After volume is created click on the 3 dots on the right side of volume you have just created. In the menu choose **Show Configuration**. This will bring up the commands you must use to make the space on the new volume available inside your VPS. You must run only the first command to format the Volume, while updaring $FILESYSTEM with ext4

![](../.gitbook/assets/image%20%2836%29.png)

_NOTE: This are just example commands. Just MUST use the commands that are displayed on your Hetzner Console._

_i.e. the command will be_

sudo mkfs._ext4_  -F /dev/disk/by-id/scsi-0HC\_Volume\_12872586

* Copy, paste and run the first command in the terminal of your VPS.

Second`,` in most cases you would have a backup folder already created on your node. If for some reason you have never done backups run the following

```text
docker exec it otnode mkdir -p ../backup
```

* Then you will need the path of your docker container which you saved at the beginning of the guide. If you did notm you must run **df -h** and save the path in the row **overlay. It should look something like this:** /var/lib/docker/overlay2/**5b0720e1a89e407a48aa3be9112f6f531ec47d7ec5662a7dafcc98506968af8d**/diff
* You will use the value you got from the **df -h** on your node. Copy the third command MOUNT VOLUME from the Instructions, while updating the second part of the command where above it says /mnt/volume-fsn1. Should look similar to the below with the updated Volume number and the node folder \(the XXXXXXX entries below\).

```text
mount -o discard,defaults /dev/disk/by-id/scsi-0HC_Volume_xxxxxxx /var/lib/docker/overlay2/xxxxxxxxxxxxxxxxxxxxxx/merged/ot-node/backup
```

* Next you run the command to perform the backup, which will go on the Volume space.

```text
docker exec otnode node scripts/backup.js --config=/ot-node/.origintrail_noderc --configDir=/ot-node/data --backupDirectory=/ot-node/backup
```

* then unmount the volume space from the VPS with the below command, updating the folder we copied at the beginning \(the xxxxxxx

```text
umount /var/lib/docker/overlay2/xxxxxxxxxxxxxxxx/merged/ot-node/backup
```

* then you need to detach the volume from the Hetzner's control panel. Go to the Volume section for this server and click on the three red dots and select Detach Volume

![](../.gitbook/assets/image%20%2837%29.png)

### Setting up the new server

Create a new VPS, selecting Ubuntu 18.04 as operating system. Dockerless nodes will not work on Ubuntu 20! Also make sure you create the server in the same datacenter as your current one, so the Volume will be accessible, otherwise it won't work.

Once you create the VPS, you need to attach the volume. Go to the Volume section on the new VPS and choose Attach, select the Volume from the dropdown and choose manual.

![](../.gitbook/assets/image%20%2842%29.png)

* Once you attach the volume, login to the VPS as root and create the backup folder in the root directory

```text
mkdir /root/backup
```

* Then mount the Volume with the command listed on the Instructions in the MOUNT VOLUME part \(third command\), while replacing the second part with the folder location, where it says /mnt/volume-fsn1 with /root/backup. Do not run commands one or two as those will wipe the backup.

![](../.gitbook/assets/image%20%2843%29.png)

i.e. should look like:

mount -o discard,defaults /dev/disk/by-id/scsi-0HC\_Volume\_12872586 /root/backup

The final step is to move the backup content to be restore ready:

```text
mv -v /root/backup/20*/* /root/backup
```

```text
mv -v /root/backup/20*/.origintrail_noderc /root/backup
```

```text
rm -rf /root/backup/20*
```

### Updating the server and installing the scripts

Run the following commands on the new VPS:

Update the server and install Git:

```text
apt update && apt upgrade -y && apt install git -yp
```

Install 1 GB SWAP

```text
fallocate -l 1G /swapfile && chmod 600 /swapfile && mkswap /swapfile && swapon /swapfile
```

```text
cp /etc/fstab /etc/fstab.bak
```

```text
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Install the scripts:

```text
git clone https://github.com/calr0x/OT-Settings.git
```

```text
git clone https://github.com/calr0x/OT-DockSucker.git
```

```text
cd OT-DockSucker
```

```text
./install-from-existing-local-volume.sh
```

The process will complete and your new dockerless would be setup. To start the node run:

```
systemctl start otnode
```

To stop the node:

```text
systemctl stop otnode
```

To restart the node:

```text
systemctl restart otnode
```

To pull the log of the node:

```text
journalctl -u otnode -f | ccze -A
```

To pull the log of the node for the past 1 hour:

```text
journalctl -u otnode -f --since "1 hour ago"
```

To pull the log of the node of jobs won for the past 24 hours:

```text
journalctl -u otnode -f --since "24 hours ago" |egrep 'cheap'\|'Accepting'\|'ve been chosen'
```

