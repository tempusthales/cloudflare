# Cloudflare Tunnels via CLI

If at any point in this walktrhough you get lost you can follow the Cloudflare Guide here: [https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/)

1. ### First download cloudflared on your machine.

```swift
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && dpkg -i cloudflared-linux-amd64.deb
```

2. ### Authenticate cloudflared

```swift
cloudflared tunnel login
```

When you do this step, you will get a URL in terminal that you need to click.  This will open a page with all the domains you have in Cloudflare and you can choose which one you will use for the tunnel.

Once you do this, it will have you logged in and given you a path where your cert will live: `/root/.cloudflared`

3. ### Create a tunnel and give it a name

```swift
cloudflared tunnel create <NAME>
```

For example: `cloudflared tunnel create demo`

This will generate your UUID for your tunnel.  It will be a long non-descript code for the tunnel you just created.  It will look something like this `a7e850d3-8960-4718-bbf2-8c0f09556a16`

All of this information will be replicated on the terminal so just keep it in mind since you will need it later on.

4. ### Create configuration file

Use nano to open `config.yml` on `/root/.cloudflared`

```swift
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
```

**For example**, using the fake UUID from above this is how the `config.yml` should look like:

```swift
tunnel: a7e850d3-8960-4718-bbf2-8c0f09556a16
credentials-file: /root/.cloudflared/a7e850d3-8960-4718-bbf2-8c0f09556a16.json
```

5. ### Start routing traffic

Now assign a CNAME record that points traffic to your tunnel subdomain

If you are connecting an **application** (like proxmox or pterodactyl)

`cloudflared tunnel route dns <UUID or NAME> <hostname>`

e.g: `cloudflared tunnel route dns panel.yourdomain.com`

If you are connecting a network add the IP/CIDR you would like to be routed through the tunnel:

e.g: (CIDR: 192.168.0.9/24 = 192.168.0.0 to 192.168.0.255)

`cloudflared tunnel route ip add <IP/CIDR> <UUID or NAME>`

e.g: `cloudflared tunnel route ip add 192.168.1.8/24 <UUID or NAME>`

You can confirm that the route has been successfully established by running:

`cloudflared tunnel route ip show`

6. ### Setup Ingress

This part is not described in the Getting Started guide in Cloudflare Docs.  I had to figure this out myself after some serious headbutting with the wall.

However they do have an [Ingress rules](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/ingress/#notlsverify) guide, which until recently it was hidden away somewhere.

So what is Ingress? Ingress is what allows you to point your tunnel or subnet.domain to a specific server.  Otherwise how else would cloudflare know where you want to point bambifawns.com to?

```swift
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json

ingress:
   - hostname: dashboard.yourdomainname.com
     service: https://your.ip.goes.here:port#
     originRequest:
        noTLSVerify: true

   - service: http_status:404
```

**Catch all rule:** `- service: http_status:404`  in case you are wondering.

7. ### Run the tunnel

Run the tunnel to proxy incoming traffic from the tunnel to any number of services running locally on your origin.

`cloudflared tunnel run <UUID or NAME>`

So for example:

`cloudflared tunnel run demo`

8. ### Run cloudflared as a service

You can install Cloudflared as a system service on Linux so you dont have to run it manually.

1. Install the `cloudflared` service: `cloudflared service install`
2. Start the service: `systemctl start cloudflared`
3. (Optional) View the status of the service: `systemctl status cloudflared`

> **If for any reason you cannot install the service.  Follow these steps:**

1. > Rename `config.yml`: `mv  /root/.cloudflared/config.yml /root/.cloudflared/config.yml.old`
2. > Uninstall the Cloudflared service by typing: `cloudflared service uninstall`
3. > Rename the config.yml again: `mv  /root/.cloudflared/config.yml.old /root/.cloudflared/config.yml`
4. > Restart Cloudflared service: `systemctl restart cloudflared`

---

# Reading and viewing material

1. > Configuring tunnel by command line - [https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/)
2. > Ingress rules - [https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/ingress/#notlsverify](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/ingress/#notlsverify)
3. > How to run as a service in Linux - [https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/as-a-service/linux/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/as-a-service/linux/)
4. > Cloudflare Tunnel Setup Guide by RaidOwl: [https://www.youtube.com/watch?v=hrwoKO7LMzk](https://www.youtube.com/watch?v=hrwoKO7LMzk)

