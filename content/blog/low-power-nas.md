---
title: Build a low power self hosted NAS server & Remote backup system with OpenMediaVault
description: Build a low power self hosted NAS server & Remote backup system with OpenMediaVault.
date: 2018-05-01
tags:
  - nas
---

Reference video: _How to build a Raspberry Pi NAS_ ([Video](https://www.youtube.com/watch?v=gyMpI8csWis))

## What do we need?
* Raspberry Pi (ours Pi 3B+, with USB 2.0)
* MicroSD
* Hard Disks to make the NAS
* Ethernet cable (optional)
> We used HDD Disks that needed an external power source because the Raspberry only can provide 1.2 A, and we needed more. We used a Dell docker to connect the disks. 
## Download Raspberry Pi Imager
1. Open the App and select the Choose OS
<img src="https://user-images.githubusercontent.com/26599092/219011119-8eba9780-44cf-430c-8d65-6a8e21b36a82.png" width="300" height="225">

2. Select other software 

<img src="https://user-images.githubusercontent.com/26599092/219011258-e2b27f61-253c-4b6d-acec-12757eb43d17.png" width="300" height="225">

3. Select the Raspberry Pi OS Lite

<img src="https://user-images.githubusercontent.com/26599092/219010886-fd4e6aa7-cb45-4b1a-96b2-17ee194aa89c.png" width="300" height="225">

4. Connect a MicroSD to your computer and choose it as generic storage.
5. Configure in Advanced options the possibility to connect via SSH, you can change the name of the Raspberry and the user if you want to.
<img src="https://user-images.githubusercontent.com/26599092/219015303-5800c878-c837-4b32-a28a-1f821f3e9238.png" width="600" height="450">

6. Select Write
## Connect to the Raspberry via SSH
The first thing is to connect to your home router and see the IP dress assigned to the Raspberry. Launch the terminal and connect to the Raspberry via SSH.
```bash
ssh user_name@IP_adress
```
If everything is correct the terminal will ask you _(...)Are you sure you want to continue connecting?_. Say yes and press return. Then you will be asked to put the password of the Raspberry that was configured in point 5 of last section.
Once you have put the password you will be logged in!
To make sure everything is updated, we can run:
```bash
sudo apt update && sudo apt upgrade
```
This command updates the repositories and installs all the available updated.
## Install OpenMediaVault
This will be the NAS software.
We recommend to do the installation connected to the router with an Ethernet cable. We did it first with wifi and the installation was cut in the middle of the process due to connection loss.

The installation process demands sudo utilization.
To download and execute the script you can use either wget or curl, feel free to use what you prefer!

_wget script_
```bash
sudo wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```
_curl script_
```bash
sudo curl -sSL https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```
Once the installation is over, you can open your usual web browser and log in into the IP address. You will see the interface of OpenMediaVault

## Installation in Debian 11
To perform the installation of OMV on a blank Debian OS, we followed the commands mentioned in ([link](https://forum.openmediavault.org/index.php?thread/39490-install-omv6-on-debian-11-bullseye/)).

First:

```bash
cat <<EOF >> /etc/apt/sources.list.d/openmediavault.list
deb http://packages.openmediavault.org/public shaitan main
# deb http://downloads.sourceforge.net/project/openmediavault/packages shaitan main
## Uncomment the following line to add software from the proposed repository.
# deb http://packages.openmediavault.org/public shaitan-proposed main
# deb http://downloads.sourceforge.net/project/openmediavault/packages shaitan-proposed main
## This software is not part of OpenMediaVault, but is offered by third-party
## developers as a service to OpenMediaVault users.
# deb http://packages.openmediavault.org/public shaitan partner
# deb http://downloads.sourceforge.net/project/openmediavault/packages shaitan partner
EOF
```

Then:
```bash
export LANG=C.UTF-8
export DEBIAN_FRONTEND=noninteractive
export APT_LISTCHANGES_FRONTEND=none
apt-get install --yes gnupg
wget -O "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc" https://packages.openmediavault.org/public/archive.key
apt-key add "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc"
apt-get update
apt-get --yes --auto-remove --show-upgraded \
    --allow-downgrades --allow-change-held-packages \
    --no-install-recommends \
    --option DPkg::Options::="--force-confdef" \
    --option DPkg::Options::="--force-confold" \
    install openmediavault-keyring openmediavault

# Populate the database.
omv-confdbadm populate

# Display the login information.
cat /etc/issue
```

If you would like to use plugins such as mergerfs you will need to install the omv-extras, since unlike the installation described for raspbian, this installation in debian does not include it, note that this is only for omv6. To do so, as mentioned in ([omv-extras wiki]( https://wiki.omv-extras.org/)), run the following command:

```bash
sudo wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | sudo bash
```

## Open Media Vault
From the reference video, ([link](https://www.youtube.com/watch?v=gyMpI8csWis)) minute 9:02,
The YouTube video uses a OpenMediaVault version older than the one we used, ours was the 6th. The interface changes a little bit but we were able to follow the steps without major problems.

**For every 1 or 2 changes, click the option of Save and Apply to set the changes!!** The changes will not be made even if you press the button "save", so press "save" and when the disclaimer yellow that indicates "Pending configuration changes" press the tick icon.

The main steps to follow are:
1. Basic configuration
- Go to, User settings/Change Password --> change the password
2.  Add a USB hard drive
- Go to, Storage/Disks --> you should see the SD card and the connected Hard Drives.To make sure everything later will run correctly, wipe the disks.

When we got to this point we had two problems, here explained and solved:
> We had a special situation, **the RaspberryPi 3+, is only able to provide 1.2A to the 4 USB ports,**  our disks were too demanding, and therefore, the Raspberry was not able to power them and we could not see them in the OpenMediaVault interface. We solved this problem connecting the disk to a dock station. and the dock station that is powered independently to the RaspBerry Pi. If your disks don't need more than 1.2A you should be save!

> The other problem we had was that we thought the Raspberry Pi could be connected to the power through a USB port on the PC, but when we connected it that way, we had problems. In the end, the most reliable way we found was to connect it directly to the power using a USB power adapter.

> The third situation we had was that when the disks were mounted, we couldn't see them in the Storage/Shared Folder dropdown menu. This issue was solved with clearing the cache of the web browser that we were using.
- Go to, Storage/File Systems --> Mount the disks using the (+) symbol, type EXT4, select the device. If you cannot see the disks in the dropdown menu, check the "special situations" that were commented before. 
-In, Storage/File Systems --> Mount the existing disks using the play button and then select the file system.
3. Create a Shared Folder
- Go to Storage/Shared Folders --> press (+), name the folder as you wish, link it to a file system (the hardware were it will be stored) and the relative path. You can set the permission as you like, add new users, and try different permissions!
4. Enabling NFS or SMB
- Go to, Services/SMB/CIFS/Settings --> enable and save 
- Go to, Services/SMB/CIFS/Shares --> click (+) Select a shared folder, tick the "Browseable" box if you want the folder and below subfolders to be shared.
- Go to, Services/NFS/Settings --> enable and save 
- Go to, Services/NFS/Shares --> click (+) Select a shared folder, set the IP address of the Clients allowed to mount the file system. Set the privilege.

With this done, you should be able to enter to your NAS with your Windows, Mac or Linux connected to the network.

## Remote Sync OpenMediaVault
After achieving this, we wanted to create a backup system for our folders.
In order to do this we followed the next video. ([link](https://www.youtube.com/watch?v=Tz8WpzsMXsQ))
