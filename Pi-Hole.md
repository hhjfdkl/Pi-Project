# Pi-Hole Setup
Set static IP through DHCP reservation
- Pretty straightforward. May vary from router to router, but try looking in the *LAN* section in the GUI
- Your server needs a static IP address for Pi-Hole to function most effectively
	* *Really, we probably should have the static IP address set due to our NAS setup also*

Install Pi-Hole: `curl -sSL https://install.pi-hole.net | bash`
- Informational menu navigation in setup. First relevant option is Upstream DNS Provider. I chose OpenDNS for now
	* There is a ton of info in the Pi-Hole documentation about which one you can choose. I'd recommend reviewing that if you're interested in making a decision based on that : https://docs.pi-hole.net/guides/dns/upstream-dns-providers/
- Do you want to install Steven's block list? : *Yes*
- Do you want to enable query logging? : *No* 
	* but this is potentially useful to enable in shared environments (like businesses) or where you'd like to troubleshoot issues.
- Select privacy mod for FTL: *Anonymous Mode*
	* Useful in conjunction with query logging
	- FTL privacy determines how much data is logged when logging is enabled
After this is completed, Pi-Hole is installed and will provide you with a generated password.
- You can and should change the password with `sudo pihole setpassword`

Sometimes it can take a minute, but Pi-Hole will automatically update its `pihole.toml` configuration file if port 80 is already in use. Just make sure port 8080 is not in use, and it will automatically rollover to 8080

> Of note, it's very easy to add pre-made block-lists into your Pi-Hole. You just need a URL to paste in the address link
- So far I am only using Steven's domain list
*May be configured more later*


## Now to set up the Pi as the default DNS server
In router, navigate to DHCP server
- Update the DNS server to match the IP address of your Pi-Hole server

**At this point, everything should be completely functional.**

## Unbound Installation
**Got this part from Crosstalk Solutions on YouTube: https://www.crosstalksolutions.com/the-worlds-greatest-pi-hole-and-unbound-tutorial-2023/**
- And he found it here in the official unbound docs: https://docs.pi-hole.net/guides/dns/unbound/
Assists Pi-Hole in looking up root DNS servers on the Internet
To install: `sudo apt install unbound -y`
Write to config file: `sudo nano -w /etc/unbound/unbound.conf.d/pi-hole.conf`
- The full text file can be found at https://docs.pi-hole.net/guides/dns/unbound/
- After the config file has been updated, we will start the unbound service is `sudo service unbound restart`
	- check to make sure it's running : `sudo service unbound status` or `systemctl status unbound`
- Then use `dig` to test that it works: `dig [address] @127.0.0.1 -p 5335` *(this uses the port that unbound is using to gather info on whatever address you type in)*

Now that we've confirmed that unbound is working, we need to configure Pi-Hole to use it
- Update Pi-Hole DNS by unchecking the box for the Upstream DNS Server previously selected and enter: `127.0.0.1#5335` in the Custom DNS servers section
- This causes us to use the *root* DNS servers for the Internet

Added Logging to Unbound by adjusting `/etc/unbound.conf.d/pi-hole.conf` to point to the default, commented out log file `/var/log/unbound/unbound.log`
- Added `log-time-ascii: yes` for human readability
- Adjusted `verbosity: 1` for now. *1 to 3 seems best*

# Issues
Per the documentation, Pi-Hole will install on 80 by default, but if that's in use it will switch to 8080. If 8080 isn't available, it will not be available and you'll have to manually configure it.
- **Read the documentation - it will save you a lot of time**
- The documentation is ~~false~~ **true** ! *(it just takes a minute to re-do the ports)* But I was able to find the .config file: `/etc/pihole/pihole.toml`

Don't add additional endpoints to your nginx redirect for Pi-Hole. If you simply redirect to `127.0.0.1:8080` it will work properly
- When I tried directing to `127.0.0.1:8080/admin/login/` it would break the UI in Chromium browsers and in Firefox the UI just wouldn't work.

Encountered an issue with `unbound-resolvconf.service`. Error message was `Failed to set DNS configuration:` and some info regarding a loopback device. 
- I found a GitHub Issue [here](https://github.com/NLnetLabs/unbound/issues/1161) that matched this exactly, and it's also in the Unbound documentation [here](https://unbound.docs.nlnetlabs.nl/en/latest/use-cases/local-stub.html). (In the docs, just search "resolvconf" and the relevant section will come up)
- I stopped, disabled, and masked the service with `sudo systemctl [stop/disable/mask] unbound-resolvconf.service`
	- After doing this, systemd doesn't automatically clear its cache, so it'll still look like you have an error. Just run `sudo systemctl daemon-reload` to make sure the daemon is refreshed and also `sudo systemctl reset-failed` to refresh systemd's records of services in the "failed" state
* It didn't seem to affect how Pi-Hole was performing, but the error message was annoying me when I saw it.
* Using the `dig` command helps to confirm Unbound is still working

# Error messages
7/16/25: Encountered error from Unbound, but it's most likely just a network hiccup and hasn't affected service from what I can tell
- `Connection error (127.0.0.1#5335): TCP connection failed while receiving payload length from upstream (Connection prematurely closed by remote server)`
- Checked unbound with `sudo netstat -nlp | grep "unbound"` and `sudo netstat -nlp | grep 5335`, it's running fine
* Deleted error log

# Notes for later additions
You can grab the password hash for your admin login from `cat /etc/pihole/setupVars.conf | grep WEBPASSWORD`
- Useful for scripting in case you need to use your admin account for something 
- It looks like it may need to be manually set up
* I read about scripting the "Disable Blocking" command and calling that through some other convenient device so you don't have to go in and manually disable blocking in case you sometimes need to access blocked sites
UI has "Basic/Expert" button. Make sure to flip to Expert to see all options for configuration
"Teleporter" is to export/import Pi-Hole's settings
