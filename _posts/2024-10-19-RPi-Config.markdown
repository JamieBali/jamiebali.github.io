---
layout: post
title:  "Raspberry Pi Config"
date:   2024-10-19 18:00:47 +0100
categories: raspberry-pi
---

This here is a list of some linux commands I've been running set set up a couple of the machines in my Raspberry Pi Cluster.
These are mostly for personal reference, but may be helpful for anyone else managing Raspberry Pi Infrastrucutre.

# Part 1 : Wireless Access Point

We can create an access point using a Raspberry Pi. There are a couple of different reasons we might want this. The common reasons I see online for why you might want this include stuff like Hotel WiFi not letting you use multiple devices, but we're using it because we want to ensure a strong and stable WiFi connection in our media room.

First, before doing anything, my RPi 4 needed some pre-work completed. This came in the form of some apt updates, and an update to the raspi-config.

{% highlight %}
sudo apt update
sudo apt upgrade
{% endhighlight %}

Once these have completed, the machine will likely need a restart, but we'll still additionally need to run the command `sudo raspi-config`. This command takes us to a window where we can scroll through settings. We needed to ensure that tha machine was fully up-to-date here or the WiFi network wouldn't connect.

Now, we can create the hotspot. This will share the ethernet connected as a wireless access point.

{% highlight %}
sudo nmcli con add type wifi ifname wlan0 mode ap con-name accesspoint ssid "SSID" autoconnect true
#=> Creates a blank WAP with the provided SSID

sudo nmcli con modify accesspoint 802-11-wireless.band bg ipv4.method shared ipv4.address IPv4
#=> Sets accesspoint to use IPv4

sudo nmcli con modify accesspoint ipv6.method disabled
#=> Disables IPv6 (RPi 4 seems to struggle with IPv6)

sudo nmcli con modify accesspoint wifi-sec.key-mgmt wpa-psk
#=> Configures the access point to use a passcode for access

sudo nmcli con modify accesspoint wifi-sec.psk "pass"
#=> Sets the passcode to the provided string

sudo nmcli con up accesspoint
#=> Activates the access point
{% endhighlight %}

Another thing to note about setting the access point up like this is that is automatically starts the access point up when the machine boots. This means nothing else needs to be done on our end when we reboot, shutdown, or move the machine.

# Part 2 : Network Attached Storage

We have a Pi 5 which is set up with a 2Tb NVMe drive attached through the PCIe port. We want to get this configured to function as a NAS so that we can have accessible storage from anywhere on our network easily. Additionally, there are some data processing scripts that we wnat to be running periodically which would be best to run from this machine.
We opted to configure the NAS using samba. The commands to configure the NAS this way are as follows:

First, we want to set up our shared location and mount the drive. Our mounted location is `/mnt/drv/nvme`.

{% highlight %}
sudo lsblk
#=> use this to find the name of the drive we're mounting into the NAS.

cd mnt
sudo mkdir drv
sudo mkfs.ext4 /dev/nvme0n1p1
sudo mount /dev/nvme0n1p1 /mnt/drv/nvme
sudo chmod -R 777 /mnt/drv/nvme
#=> use this to mount the drive into the root and set the perms
{% endhighlight %}

Once the mount is setup, we can install and configure Samba.

{% highlight %}
sudo apt install samba samba-common-bin
#=> use this to install the samba application

sudo nano /etc/samba/smb.conf
#=> now we need to configure the Samba application
{% endhighlight %}

The config for samba is as follows:

{% highlight %}
path = "path to install"
writeable = yes
create mask = 0775
directory mask = 0775
{% endhighlight %}

Once this is done, we need to restart the samba filesystem by running `sudo systemctl restart smbd`

Lastly, we need to grant access to remote into this fileshare. We can do this as follows by creating a user and granting them a Samba password. Both Usernames and Passwords are case-sensitive here.

{% highlight %}
sudo adduser "username"
sudo smbpasswd -a "username"
{% endhighlight %}

Now that everything is configured, we can remotely access the NVMe drive. You can run `hostname -I` to get the hostname of the machine which you will need to UNC path to the machine for access.