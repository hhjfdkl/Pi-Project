# nginx setup

Since I want to host multiple applications on my Pi, I want to use nginx reverse proxy. This is much more flexible than adjusting and keeping track of the ports on my Pi applications.

## Steps
1. Install nginx : `sudo apt install nginx`
2. Create nginx configuration for applications
** NOTE ** : You enable/disable sites in the /etc/nginx/sites-enabled/ directory: You place symlinks to the .conf files present inside the sites-available/ directory. To disable, simply delete the symlink. 
	- Open the nginx configuration file : `sudo nano /etc/nginx/sites-available/reverse-proxy.conf` *(likely creating the reverse-proxy.conf file)*
		- *This should be based on  the "default" file that's present in sites-available* and it should look like the below:
```
server {
    listen 80;  # Port 80 (HTTP) 
    server_name raspberrypi.local;  # Replace with your Pi's hostname or domain name

    # Proxy for app1 
    location /app1/ {
        proxy_pass http://127.0.0.1:8080/;  # Local IP and port for app1
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Proxy for app2
    location /app2/ {
        proxy_pass http://127.0.0.1:8081/;  # Local IP and port for app2
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- The above creates forwarding paths to `/app1` and `/app2` when the server receives requests on port 80. Essentially, when we get a request to `pi-ip-address` or `server_name`, nginx will check which endpoint to forward the traffic to.
- Additionally, you can add another location to `/` in the same format to provide default behavior for the reverse proxy : So you could forward traffic to app2's port at the `/` endpoint
- For any configurations beyond what is above, see the default config file for reference. *(here, we're only doing HTTP, I may return for SSL at a later point)*

3. Create a symlink in `sites-enabled/` : `sudo ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/`
4. After making your changes, restart nginx : `sudo systemctl restart nginx`

## Additional notes and issues
There's additional setup we could do for TLS/SSL, but for now I'm just going to keep it to HTTP
In the `location` blocks above, the `set_header` commands fill out the HTTP request that nginx passes to the backend service. So it's like we're taking what we received and building out a new HTTP request, and sending it out to the `127.0.0.1:{whatever}` ip address *(which is our loopback address)*
- Also of note, `$scheme` indicates whether the request was http or https

You can test your nginx.conf (which will test all of your config files) with `sudo nginx -t`
You can also check the logs in `/var/log/nginx/`
The reverse proxy didn't work with `/app1` until I added an addition `/` at the end, and also added a `/` after the port number in the `proxy_pass` address. 
- So `location /app1/` plus `proxy_pass http://127.0.0.1:8080/` worked for me.
- Updated example above
