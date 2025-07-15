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

7/14/25 : Easier to update the port just by using the GUI : `System > Workbench > Port`
- Had conflicts with other services I was trying to set up and found it was more straightforward to just use the GUI
