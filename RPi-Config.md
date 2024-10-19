---
layout: page
title: Raspberry Pi Config
permalink: /RPi
---

Storage page primarily. Will update shortly and include meanings.

    1  lsblk
    2  cd ../
    3  mkdir ./mnt
    4  cd mnt
    5  ls
    6  mkdir drv
    7  sudo mkdir drv
    8  sudo mount /dev/mmcblk0p2 /mnt/drv
    9  sudo mkdir /mnt/drv/shared
   10  sudo chmod -R 777 /mnt/drv/shared
   11  sudo apt install samba samba-common-bin
   12  sudo nano /etc/samba/smb.conf
   13  sudo systemctl restart smbd
   14  sudo adduser Jamie
   15  sudo adduser pi-nas-user
   16  sudo smbpasswd -a username
   17  sudo smbpasswd -a Jamie
   18  sudo apt install samba samba-common
   19  sudo smbpasswd -a Jamie
   20  sudo chmod -R 775 /mnt/drv/shared
   21  sudo nano /etc/samba/smb.conf
   22  sudo systemctl restart smbd
   23  suda apt update
   24  sudo apt update
   25  sudo apt upgrade
   26  sudo rmdir /mnt/drv
   27  sudo systemctl down smbd
   28  sudo systemctl stop smbd
   29  sudo rmdir /mnt/drv
   30  sudo rmdir /mnt/drv/shared
   31  sudo rmdir /mnt
   32  sudo rmdir /mnt/drv
   33  sudo unmount /mnt/drv
   34  sudo umount /mnt/drv
   35  sudo rmdir /mnt/drv
   36  mkdir /mnt/drv
   37  sudo mkdir /mnt/drv
   38  sudo mkdir /mnt/drv/shared
   39  chown Jamie /mnt/drv
   40  sudo chown Jamie /mnt/drv
   41  sudo chown Jamie /mnt/drv/shared
   42  sudo lsblk
   43  sudo nano /etc/samba/smb.conf
   44  sudo smbpasswd -a Jamie
   45  sudo systemctl restart smbd
   46  hostname -I
   47  sudo smbpasswd -a Holly
   48  sudo adduser Holly
   49  sudo adduser HollyS
   50  sudo adduser Holly123
   51  sudo adduser Holly --allow-bad-names
   52  sudo smbpasswd -a Holly
   53  raspi-config
   54  sudo raspi-config
   55  hostname -I
   56  lsblk
   57  sudo lsblk -f
   58  sudo fdisk /dev/nbme0n1
   59  sudo fdisk /dev/nvme0n1
   60  lsblk
   61  sudo mkdir /mnt/drv/nvme
   62  sudo chown Jamie /mnt/drv/nvme
   63  sudo mount /dev/nvme0n1p1 /mnt/drv/nvme
   64  sudo mount /dev/nvme0n1 /mnt/drv/nvme
   65  dmesg
   66  sudo mount /dev/nvme0n1 /mnt/drv/nvme
   67  mkfs.ext4 /dev/nvme0n1p1
   68  sudo mkfs.ext4 /dev/nvme0n1p1
   69  sudo mount /dev/nvme0n1 /mnt/drv/nvme
   70  sudo mount /dev/nvme0n1p1 /mnt/drv/nvme
   71  sudo nano /etc/samba/smb.conf
   72  sudo systemctl restart smbd
   73  lsblk -f
   74  sudo lsblk -f
   75  ls
   76  cd ../
   77  cd mnt
   78  cd drv
   79  cd nvme
   80  ls
   81  cd lost+found
   82  sudo cd lost+found
   83  sudo rmdir lost+found
   84  ls
   85  sudo apt list -u
   86  sudo nmcli con add type wifi ifname wlan0 mode ap con-name accesspoint ssid "Pissbaby-WAPMAN" autoconnect true
   87  sudo nmcli con modify accesspoint 802-11-wireless.band bg ipv4.method shared ipv4.address 192.168.6.1/24
   88  sudo nmcli con modify accesspoint ipv6.method disabled
   89  sudo nmcli con modify accesspoint wifi-sec.key-mgmt wpa-psk
   90  sudo nmcli con modify accesspoint wifi-sec.psk "SweetPrincess"
   91  sudo nmcli con up accesspoint
   92  sudo nmcli c
   93  sudo apt update
   94  sudo apt list -u
   95  sudo raspi-config
   96  sudo nmcli con down accesspoint
   97  sudo nmcli con up accesspoint
   98  journalctl | grep wifi
   99  journalctl | grep hotspot
  100  journalctl | grep accesspoint
  101  sudo nmtui
  102  nmcli
  103  nmcli d
  104  nmcli c
  105  nmcli c delete accesspoint
  106  sudo nmcli c delete accesspoint
  107  nmcli c
  108  nmcli c delete Pissbaby-WAPMAN
  109  sudo nmcli c delete Pissbaby-WAPMAN
  110  nmcli c
  111  nmcli d
  112  nmcli d wifi
  113  nmcli c
  114  sudo nmcli con add type wifi ifname wlan0 mode ap con-name "WAPMAN" ssid "Pissbaby-WAPMAN" autoconnect true
  115  nmcli c
  116  sudo nmcli modify WAPMAN 802-11-wireless.band bg ipv4.method shared
  117  sudo nmcli con modify WAPMAN 802-11-wireless.band bg ipv4.method shared
  118  sudo nmcli con modify WAPMAN ipv6.method disabled
  119  sudo nmcli con modify WAPMAN wifi-sec.key-mgmt wpa-psk
  120  sudo nmcli con modify WAPMAN wifi-sec.psk "SweetPrincess"
  121  sudo nmcli con
  122  sudo nmcli con up WAPMAN
  123  turnoff
  124  poweroff
  125  sudo poweroff
  126  raspi-config
  127  sudo raspi-config
  128  sudo lsblk
  129  sudo nano /etc/samba/smb.conf
  130  cd ../
  131  cd etc
  132  cd mnt
  133  ls
  134  cd ../
  135  ls
  136  cd mnt
  137  ls
  138  cd ../
  139  sudo lsblk
  140  cd nvme0n1
  141  cd /dev/nvme0n1
  142  cd mnt
  143  ls
  144  cd drv
  145  ls
  146  cd nvme
  147  ls
  148  lsblk
  149  cd ../
  150  findmnt
  151  cat
  152  mount
  153  sudo mount /dev/nvme0n1p1 ./nvme
  154  cd ../
  155  sudo nano /etc/rc.local
  156  sudo reboot
  157  ssh Jamie@192.168.0.91
  158  sudo nmcli
  159  sudo nmcli con
  160  sudo nmcli con modify WAPMAN ssid "PiFi"
  161  sudo nmcli con modify down WAPMAN
  162  sudo nmcli con modify stop WAPMAN
  163  sudo nmcli con down WAPMAN
  164  sudo nmcli con up WAPMAN
  165  sudo nmcli con modify WAPMAN 802-11-wireless.band a
  166  sudo nmcli con down WAPMAN
  167  sudo nmcli con up WAPMAN
  168  history