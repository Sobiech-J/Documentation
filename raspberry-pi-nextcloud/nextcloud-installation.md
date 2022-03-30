# Install and Configure Nextcloud Snap

## Install Nextcloud snap

**Set Up External Storage**
* If an external hard drive will be used to hold nextcloud's data, make sure a folder is created to hold it and that all permissions are set to root:
  * Create data folder: `sudo mkdir -p /mnt/data-drive/nextcloud/data`
  * set owner and group to root: `sudo chown -R root:root /mnt/data-drive/nextcloud/data`
  * change permissions: `sudo chmod 770 /mnt/data-drive/nextcloud/data`

**Install Nextcloud Snap**
* install package: `sudo snap install nextcloud`
* create admin account: `sudo nextcloud.manual-install <admin-username> <password>`

**If you are using external storage**
* allow connection to external media if you are using an external hard drive: `sudo snap connect nextcloud:removable-media`
* `sudo snap stop nextcloud`
* copy nextcloud data directory to external hard drive (if this is the first time you are setting up nextcloud and you dont already have data): `sudo cp -r /var/snap/nextcloud/common/nextcloud/data /mnt/data-drive/nextcloud/data`
  * may need to delete and folder already named /mnt/data-drive/nextcloud/data before doing this.
* Change where nextcloud is keeping its data directory: `sudo vim /var/snap/nextcloud/current/nextcloud/config/config.php`
  * change line with `'datadirectory' =>` to `'datadirectory' => /mnt/data-drive/nextcloud/data`
* restart nextcloud: `sudo snap start nextcloud`


**Configure Trusted Domains**
* `sudo snap run nextcloud.occ config:system:set trusted_domains 1 --value=<your-server's-ip-adress>`
* `sudo snap restart nextcloud`


**Configure SSL**
* allow ports 80 and 443 in firewall: `sudo ufw allow 80, 443/tcp`
* if you have a domain name to use: `sudo nextcloud.enable-https lets-encrypt`
* if you do not have a domain name or are fine with a self-signed certificate: `sudo nextcloud.enable-https self-signed`
  * cert key can be found in /var/snap/nextcloud/current/certs/self-signed. Active one is in /var/snap/nextcloud/current/certs/live


**Change ports for Nextcloud**
* `sudo snap set nextcloud ports.http=<new-port>`
* `sudo snap set nextcloud ports.https=<new-port>`
* if behind a proxy: `sudo nextcloud.occ config:system:set overwritehost --value="example.com:<new-port>"`

**Backup Nextcloud Data**
* connect second hard drive and mount it the same way as the first
* `sudo mkfs.ext4 /dev/sdb` if it needs a new filesystem
* `sudo mount /dev/sdb /mnt/backup-drive`
* backup data with: `sudo rsync -avr /mnt/data-drive/nextcloud/data /mnt/backup-dive/nextcloud`
* umount backup disk: `sudo umount /dev/sdb`
