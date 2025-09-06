📌 Prerequisites

Ensure the EBS volume and EC2 instance are in the same Availability Zone.

The EC2 instance should be in a running state.

AWS Documentation

⚠️ Important Considerations

Data Preservation: If you're attaching a volume that contains data, do not format it. Instead, mount it directly after verifying its filesystem.

Multi-Attach: If you need to attach a single EBS volume to multiple EC2 instances simultaneously, consider using EBS Multi-Attach with io1 or io2 volume types. This feature is available in specific regions and has certain limitations. 
AWS Documentation

🛠️ Step-by-Step Guide
1. Attach the EBS Volume

Using the AWS Management Console:

Navigate to the EC2 Dashboard
.

In the left navigation pane, choose Volumes.

Select the volume you want to attach.

Click Actions > Attach Volume.

In the dialog box:

For Instance, select the EC2 instance.

For Device, specify a device name (e.g., /dev/xvdf).

Click Attach Volume.

1. Verify the Attached Volume

List block devices:
```bash
lsblk
```

You should see the new volume (e.g., /dev/xvdf) listed.

2. Create a Filesystem on the Volume

Check if the volume is empty:
```bash
sudo file -s /dev/xvdf
```

If it returns data, the volume is empty. Proceed to format:
```bash
sudo mkfs -t ext4 /dev/xvdf
```

Note: Formatting will erase all data on the volume.

DevOpsCube

5. Mount the Volume

Create a mount point and mount the volume:
```bash
sudo mkdir /mnt/ebs_volume
sudo mount /dev/xvdf /mnt/ebs_volume
```
6. Configure Automatic Mount on Reboot

To ensure the volume mounts automatically after a reboot:

Retrieve the UUID of the device:
```bash
sudo blkid /dev/xvdf
```

Edit the /etc/fstab file:
```bash
sudo nano /etc/fstab
```

Add the following line at the end (replace <UUID> with the actual UUID):

UUID=<UUID> /mnt/ebs_volume ext4 defaults,nofail 0 2


Save and exit the editor.

Note: It's recommended to back up the /etc/fstab file before editing.


---
2)Step-by-Step Debugging (AWS EC2):
#1. Check if the volume is visible at all

Run:

```bash
lsblk
```

#"(Do you see /dev/xvdf in the output?)"
(or)
/dev/nvme

#You're likely using a Nitro-based EC2 instance, and EBS volumes appear as /dev/nvme... instead of /dev/xvdf.

(Then format and mount it):
bash
# Replace nvme1n1 with your actual device name

#(danger:read here)
# "it will format the disk data: 1st time we need run but second time if format is require only we should run."

format the data
```bash
sudo mkfs -t ext4 /dev/nvme1n1
```
create directory for volume
```bash
sudo mkdir -p /mnt/myvolume    
```
volume mount to server to use it
```bash
sudo mount /dev/nvme1n1 /mnt/myvolume
```

(Make it Mount on Reboot):
#before that Backup the /etc/fstab file for safeside.
```bash
sudo cp /etc/fstab /etc/fstab.bak
```

Add to /etc/fstab:
bash
```bash
echo "/dev/nvme1n1 /mnt/myvolume ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
```
#Final Reminder:
Stop trying to use /dev/xvdf manually if you're on a Nitro-based EC2 instance — use the actual device name (nvme1n1, etc.) as shown by lsblk.

(Mount Command):

#Create a mount point (directory):
```bash
sudo mkdir -p /mnt/myvolume
```
#Mount the device (example uses /dev/nvme1n1):
```bash
sudo mount /dev/nvme1n1 /mnt/myvolume
```
#"Replace /dev/nvme1n1 with your actual device (you can find it using lsblk)."

(To Check Mount):
```bash
df -hT
```
(Unmount Command):

#To unmount the volume:
```bash
sudo umount /mnt/myvolume
```
#If the volume is busy (used by a process), you may need to stop that process first or use:
```bash
sudo fuser -vm /mnt/myvolume
```
#To force unmount:
```bash
sudo umount -l /mnt/myvolume
```
