# Radarr Build
This recipe is for setting up Radarr. The following assumes your are familiar with Radarr and its preference menus and configuration options.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild).
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building).
- [x] pfSense is fully configured as per [HAProxy in pfSense](https://github.com/ahuacate/proxmox-reverseproxy/blob/master/README.md#haproxy-in-pfsense)
- [x] Deluge LXC with Deluge SW installed as per [Deluge LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#40-deluge-lxc---ubuntu-1804).
- [x] NZBGet LXC with NZBGet SW installed as per [NZBget LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#300-nzbget-lxc---ubuntu-1804)
- [x] Jackett installed as per [Jackett LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#500-jackett-lxc---ubuntu-1804)
- [x] Radarr LXC with Radarr SW installed as per [Radarr LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#90-radarr-lxc---ubuntu-1804)

Tasks to be performed are:
- [ ] 1.00 Easy Radarr Configuration
- [ ] 2.00 Manually Configure Radarr Settings
- [ ] 3.00 Create & Restore Radarr Backups
- [ ] 00.00 Patches & Fixes


## 1.00 Easy Radarr Configuration
You have two choices to configure Radarr - the scripted Easy Method or manually enter all the settings yourself. If you choose the manual route proceed to Step 2 [HERE](https://github.com/ahuacate/radarr/blob/master/README.md#200-manually-configure-radarr-settings).

The scripted Easy Method uses our pre-built configuration files which includes everything except three key settings:

*  Adding your NZB Usenet Indexer provider accounts which can be done by performing this step [2.03 (B) Configure Indexers](https://github.com/ahuacate/radarr/blob/master/README.md#203-configure-indexers)
*  Updating Deluge access login username & password if you are not using the default credentials (i.e pwd=deluge) [2.04 (A) Configure Download Client](https://github.com/ahuacate/radarr/blob/master/README.md#204-configure-download-clients)
*  Updating Radarr to use a secure login username & password which can be done by performing this step [2.06 Configure General](https://github.com/ahuacate/radarr/blob/master/README.md#206-configure-general); *and,*

Begin with the Proxmox web interface and go to `typhoon-01` > `115 (Radarr)` > `>_ Shell` and type the following:
```
sudo systemctl stop radarr.service &&
sleep 5 &&
rm -r /home/media/.config/Radarr/nzbdrone.db &&
wget https://raw.githubusercontent.com/ahuacate/radarr/master/backup/nzbdrone.db -O /home/media/.config/Radarr/nzbdrone.db &&
wget https://raw.githubusercontent.com/ahuacate/radarr/master/backup/config.xml -O /home/media/.config/Radarr/config.xml
chown 1005:1005 /home/media/.config/Radarr/nzbdrone.db &&
chown 1005:1005 /home/media/.config/Radarr/config.xml &&
sudo systemctl restart radarr.service
```

Thats it. Now go and complete Steps 2.03 (B), 2.04 (A) and 2.06.


## 2.00 Manually Configure Radarr Settings
Browse to http://192.168.50.116:7878 and login to Radarr. Click the `Settings Tab` and click `Advanced Settings` to set `Shown` state. Configure all your tabs as follows.

### 2.01 Configure Media Management
![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/media_management.png)

### 2.02 Configure Profiles
Edit Delay Profiles. Add 300 minutes to the torrent delay.
![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/profiles.png)

### 2.03 Configure Indexers
This is where you configure Radarr to use Usenet a your primary search indexer and Torrents as secondary. For torrents Radarr uses Jackett which must be installed as shown [HERE](https://github.com/ahuacate/jackett).

**A) Add Jackett as a Indexer**

Create a new torrent indexer using the `Torznab Jackett Preset` template and fill out the details as shown below.

| Add Torznab | Value
| :---  | :---:
| Name | `Jackett`
| Enable RSS Sync | `No`
| Enable Search | `Yes`
| URL | `http://192.168.30.113:9117/torznab/all/api`
| Multi Languages | leave blank
| API Key | `s9tcqkddvjpkmis824pp6ucgpwcd2xnc`
| Categories | `2000,2010,2020,2030,2035,2040,2045,2050,2060`
| Anime Categories | leave blank
| Additional Parameters | leave blank
| Remove year from search string | No
| Search by Title | `Yes`
| Minimum Seeders | `1`
| Required Flags | leave blank

And click `Save`. The finished Jackett configuration looks like:

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/torznab.png)

**B) Add Usenet Indexers**

Add all your Usenet indexers providers with the `Newsnab` presets (or custom if your provider is not listed).

Finally edit the `Options` Retention to `1500` days.

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/indexers.png)

### 2.04 Configure Download Clients
**A)  Deluge Download Client**

First create a new download client using the `Torrent > Deluge` template and fill out the details as shown below.

| Add Deluge | Value | Notes
| :---  | :---: | :---
| Name | `Deluge`
| Enable| `Yes`
| Host | `192.168.30.113`
| Port | `8112`
| URL Base| leave blank
| Password| `insert your deluge password` | This is your Deluge login password.
| Category | `radarr-movies`
| Recent Priority | First
| Older Priority | Last
| Add Paused | No
| Use SSL | No

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/deluge.png)

**B)  NZBGet Download Client**

First create a new download client using the `Usenet > NZBGet` template and fill out the details as shown below.

| Add NZBGet | Value | Notes
| :---  | :---: | :---
| Name | `NZBGet`
| Enable| `Yes`
| Host | `192.168.30.112`
| Port | `6789`
| URL Base| leave blank
| Username | `client`
| Password| `insert your client password` | This is your NZBGet client password.
| Category | `radarr-movies`
| Recent Priority | Normal
| Older Priority | Normal
| Add Paused | No
| Use SSL | No

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/nzbget.png)

Other `download tab` settings must be set as follows:

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/download_client.png)

### 2.05 Configure Connect
Create a new connection using the `Emby (Media Browser)` template and fill out the details as shown below.

| Add - Emby (Media Browser) | Value | Notes
| :---  | :---: | :---
| Name | `Jellyfin`
| On Grab | `No`
| On Download | `Yes`
| On Upgrade | `Yes`
| On Rename | `Yes`
| Filter Movie Tags | leave blank
| Host | `192.168.50.111` 
| Port | `8096`
| API Key | Insert your Jellyfin API key | *Note, create one in Jellyfin specially for Radarr*
| Send Notifications| `Yes`
| Update Library | `Yes`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/jellyfin.png)

### 2.06 Configure General
Here are required edits: 1) URL Base; and, 2) setting the security section to enable username and login.

| Start-Up | Value | Notes
| :---  | :---: | :---
| Bind Address | `*`
| Port Number | 7878
| URL Base | `/radarr`
| Enable SSL | No
| Open Browser on start | Yes
| **Security**
| Authentication | `Basic (Browser Pop-up)`
| Username | `storm` | *Note, or whatever username you choose*
| Password | `insert password here` | *Add a complex password and record it i.e oTL&9qe/9Y&RV*
| API Key | leave default

And click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/general.png)

### 2.07 Configure UI
![alt text](https://raw.githubusercontent.com/ahuacate/radarr/master/images/ui.png)


## 3.00 Create & Restore Radarr Backups
Radarr has a built in backup service. Radarr will execute a backup every 7 days creating a zip file located in `/home/media/.config/Radarr/Backups/scheduled`.

But it's good idea to make a raw backup of your working base settings configuration, including all settings, before adding any movie media. This backup file must be stored outside of the Radarr CT container for safe keeping. Then in the event of needing to recreate a Radarr CT you can use this backup file to quickly restore all your Radarr settings. This backup file must be named `radarr_backup_base_settings.zip` and be located on your NAS in folder `/mnt/backup/radarr` for below scripts to work.

### 3.01 Create a Base Settings Backup
After you have completed Steps 1.00 or Steps 2.00 its time to create **new** private base settings backup. This file will be stored on your NAS for future rebuilds when and if required.

Browse to http://192.168.50.116:7878 and login to Radarr. Click the `Systems Tab` > `Logs Tab` > `Table/Files/Updates Tabs` and click `Clear Logs` on all.

Then click `System Tab` > `Backup Tab` and click `Backup` to create a new backup file which will be shown with a name like `radarr_backup_2019.09.24_05.39.55.zip`. Now right click on this newly created file (at the top of list) and save to your NAS share `/proxmox/backup/radarr` (locally mounted as /mnt/backup/radarr). Rename your backup file `radarr_backup_2019.09.24_05.39.55.zip` to `radarr_backup_base_settings.zip`.

### 3.03 Restore to Radarr Base Settings
With the Proxmox web interface go to `typhoon-01` > `116 (radarr)` > `>_ Shell` and type the following:
```
sudo systemctl stop radarr.service &&
sleep 5 &&
rm -r /home/media/.config/Radarr/nzbdrone.db* &&
unzip -o /mnt/backup/radarr/radarr_backup_base_settings.zip 'nzbdrone.db*' -d /home/media/.config/Radarr &&
chown 1005:1005 /home/media/.config/Radarr/nzbdrone.db* &&
sudo systemctl restart radarr.service
```

### 3.03 Restore the lastest Radarr backup
If you want to restore to your last backup (this backup is a maximum of 7 days of age) use the Proxmox web interface and go to `typhoon-01` > `116 (radarr)` > `>_ Shell` and type the following: 
```
sudo systemctl stop radarr.service &&
sleep 5 &&
rm -r /home/media/.config/Radarr/nzbdrone.db* &&
newest=$(ls -t /home/media/.config/Radarr/Backups/scheduled/*.zip | head -1) &&
echo $newest &&
unzip -o "$newest" 'nzbdrone.db*' -d /home/media/.config/Radarr &&
chown 1005:1005 /home/media/.config/Radarr/nzbdrone.db* &&
sudo systemctl restart radarr.service
```

## 00.00 Patches & Fixes
