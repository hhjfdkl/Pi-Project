# Raspberry Pi NAS





## Hardware Used
Machine: Raspberry Pi 5, 4GB RAM
Storage: WD MyPassport
- USB-powered
Power Supply: CanaKit 45W
Chasis: CanaKit Case for Raspberry Pi 5
- Optional
Cooling: Raspberry Pi Active Cooler
SD Card: MicroSD Card 32GB
- Also, a USB adapter to flash the OS onto the card
Network cable: Cat5/Cat6 cable

## Software
Operating System: Raspberry Pi OS 64-bit Lite
Raspberry Pi Imager: To flash OS onto the microSD card
Hosted Application: OpenMediaVault (OMV)


## Steps for Basic Setup
Hardware Assembly
1. Assemble Pi in the case
2. Connect cables

Put image onto MicroSD
1. Plug in the MicroSD with your USB adapter
2. Use Raspberry Pi Imager
3. Select Raspberry Pi 5 for Device
4. Select Raspberry Pi OS Lite for Operating System
5. Select the MicroSD for Storage 
	- Review the device and make sure it's the correct one

Put the imaged microSD card into the pi, power it on and connect it to your router via cable

## Steps for OpenMediaVault setup
1. ssh into the pi : `ssh <user>@<ip-address>`
	* To find the Pi's IP address, either use your router's console or scan your network with something like *nmap*
	- Enter any passwords you designated during OS setup
	- If not designated, "raspberry" should be the default
2. Update the package manager and then install updates : `sudo apt update && sudo apt upgrade`
3. Install OMV : `wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash`
	* This will take a few minutes
4. Once the OMV installation finishes, the pi will reboot

Once OMV is installed, it can be accessed through the browser UI. Just type the Pi's ip address into a browser window and you'll be there
1. Change the default credentials of the admin user
2. Plug in a hard drive, make sure OMV recognizes it.
	- The UI should show the microSD card the OS is on, and the external plugged in to the USB port
3. Mount the hard drive in File Systems
	- click the "Play" button *(Mount an existing file system)*
	* For each OMV step, you'll need to apply changes. After you apply changes from mounting the file system, the UI will show storage space available and used on the device
4. After applying changes, create a shared folder to put the mounted file system in.

Now that the drive is mounted and a shared folder is created, we can set up sharing on our network in the *Services* section of OMV
1. NFS > Settings > Enable (Select any specific versions desired)
2. NFS > Shares > Create, select a shared folder
	* Client : specify the IP address range to use here. 
	ex: 192.168.128.0/24 if you want to specify all local ip addresses from 192.168.128.0 to 192.168.128.255
* The above is true for SMB as well
	- More options with SMB, but it will work at baseline if only SMB enabled and adding the share with default settings.
* Use SMB for Windows-based file systems (NTFS) and NFS for 

## Issues encountered
7/12/25: Don't try to use NFS with NTFS. SMB works with NTFS easily.
- If you want to use NFS, format your hard drive as ext4. In my case, I'm using a prior external hard drive that was originally used with a Windows machine, so it's NTFS.

7/12/25: When setting up your shared folder, *make sure you pay attention to the relative path*
- I had set "Test1" as my original share name, and it pre-populates the relative path to match. This created an empty directory called "Test1" in my hard drive and shared that on the network.

## Change Notes
7/12/25 : Only conducted the basic set up. I'll document any future changes or tinkering on this Pi here. 
- Enabled monitoring, set up the dashboard.
- Clock was out of sync. Corrected through `timedatectl` on Pi and in `System > Date & Time` in OMV
- Pi-Hole installation next? Will need to use a different port than OMV to avoid conflicts

7/13/25 : 
- Add in nginx reverse proxy to handle routing of traffic between Pi-Hole and OMV

7/14/25 :
- nginx Reverse proxy setup **completed** : See additional document added.
	* First-time use of nginx and learned a lot about setting up this basic config in this.
	- Adjusted omv to run on 8080 with the endpoint `/omv/` in the nginx reverse proxy
	- pi-hole on 80, is set to be `/pihole/` and `/`, so when calling the Pi, it will open the Pi-Hole menu
- pi-hole setup

7/15/25 :
- TODO: Break up this README into different spaces - use the README as an index and have the documents referenced in the index instead of how I have it set up now
