# Ubuntu/Raspberry Pi Set Up

**Install Ubuntu on Raspberry Pi**
* Install Raspberry Pi Imager
  * Works best on Mac. Windows 10 and 11 have some kind of driver bug with some USB Micro SD readers.
* Under Operating System select the Long Term Support (LTS) version of Ubuntu Server.
* Select the Micro SD card under Device
* Click "Write"


**Change Username and Password**
* Login to to fresh Ubuntu install on Raspberry Pi
  * default name, device and password are all "ubuntu"
  * you will be prompted to change the password on the default ubuntu account

Change Username:
* Create a second sudo account called temporary
  * `sudo adduser temporary`
  * `sudo adduser temporary sudo`
* Logout and log back in as the temporary, then change the original ubuntu account's name and its group name
  * `sudo usermod -l new-name ubuntu`
  * `sudo usermod -d /home/new-name -m new-name`
  * `sudo groupmod --new-name new-name ubuntu`
* Login with the original user with its new username, then delete the temporary user
  * `sudo deluser --remove-home temporary`


**Change Hostname**
* `sudo vim /etc/hostname`
* you may have to use `sudo vim /etc/hosts` as well, but usually not.
* `sudo reboot`

**Enable Wifi**
* If you already have ethernet access:
  * Install network manager package: `sudo apt install network-manager`
  * Enable wifi: `nmcli r wifi on`
  * List wifi connections: `nmcli d wifi list`
  * Sign into wifi: `nmcli d wifi connect <wifi-name> password <password>`

* Or, if you lack ethernt connection download network manager or if the above does not work:
  * find the name of the wifi adapter file (may be called wlan0): `ls /sys/class/net`
  * find wifi config file: `ls /etc/netplan`
  * change wifi config file found with the last command (it was 50-cloud-init.yml on the ARM processor Ubuntu Server 20.04 LTS install): `sudo vim /etc/netplan/50-cloud-init.yml`
  * see this [LinuxHint guide](linuxhint.com/configure-wifi-ubuntu/) to altering the config file
  * check the config file: `sudo netplan --debug apply`
  * `sudo reboot`
  * Check connections: `ip a`


**Update & Upgrade**
* `sudo apt update`
* `sudo apt upgrade`


**Set Up Remote Login That Requires a SSH Private Key**
* Create a ed25519 key pair: `ssh-keygen -t ed25519`
  * press enter three times for default settings (no passkey and default location in home/username/.ssh/)
* Put public key in authorized_keys to allow ssh in with the private key: `cat id_ed25519.pub >> authorized_keys`
* Login to server with another device over sftp to get the private key:
  * `sftp username@servername`
  * `get .ssh/id_ed25519`
* Return to server and only allow ssh logins with the private key:
  * `sudo vim /etc/ssh/sshd_config`
  * find the line "PasswordAuthentication yes" and change to "PasswordAuthentication no"
  * `sudo systemctl restart ssh` or `sudo reboot`


**Mount an External Drive**
* plugin external drive
* find list of devices: `sudo fdisk -l`
  * we'll use "/dev/sda" as the device in the example
* make a directory that will be mounted: `sudo mkdir /mnt/data-drive`
* format the drive as ext4: `sudo mkfs.ext4 /dev/sda`
* mount the drive: `sudo mount /dev/sda /mnt/data-drive`
* add device to /etc/fstab if the device will always be plugged in: `sudo vim /etc/fstab`
  * get device's UUID with `sudo blkid`
  * create a new line in /etc/fstab with all the following information about the external drive seperated by tabs: `UUID="your-drives-UUID" /mnt/data-drive ext4 defaults 0 2`
  * check if /etc/fstab is correct by unmounting drive then automounting. Put a file in /mnt/data-drive first so you can see when it is mounted and when it is not: `sudo umount /mnt/data-drive` then `sudo mount -a` then `ls /mnt/data-drive`