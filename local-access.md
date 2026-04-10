# Accessing Self Hosted Services Remotely

When the exit node is active all traffic routes through your VPN. This 
includes requests to your self hosted services which means using a LAN IP 
like `192.168.1.x` will not work — that address is not reachable through the 
VPN tunnel.

There are two ways to solve this depending on whether your services have a 
custom domain or not.

## MagicDNS for services without a custom domain

Tailscale assigns every device on your tailnet a hostname via MagicDNS. Your 
server gets a name like `servername.yourtailnetname.ts.net` that is always 
reachable from any Tailscale device regardless of location or whether the exit 
node is active.

To use this:

1. Open the Tailscale app on your device
2. Make sure **Allow local network access** is enabled in the exit node settings
3. Access your services using the MagicDNS hostname instead of the LAN IP

So instead of bookmarking `http://192.168.1.x:8080` which only works at home, 
bookmark `http://servername:8080` or 
`http://servername.yourtailnetname.ts.net:8080` which works anywhere. One time 
change, works consistently across all locations and network states.

Find your server's MagicDNS hostname at 
`https://login.tailscale.com/admin/machines` — it is listed under each device.

## Split DNS for custom domain based services

If your services are accessible via a custom domain like `service.yourdomain.com` 
you need Split DNS. Without it a device on the exit node resolves your domain 
publicly — the request goes out through the VPN to the internet, hits your 
domain's public DNS, and routes back. This adds unnecessary latency and can 
fail if your service is not publicly exposed.

Split DNS tells Tailscale — for this specific domain, send DNS queries directly 
to my server via the Tailscale tunnel instead of resolving publicly.

Go to `https://login.tailscale.com/admin/dns` and scroll to Nameservers. 
Click Add nameserver and select Custom. Enter your server's Tailscale IP — 
the `100.x.x.x` address from the machines page. Check Restrict to domain and 
enter your root domain, for example `yourdomain.com`. Save.

Using the root domain covers all subdomains automatically. A single entry for 
`yourdomain.com` handles `service1.yourdomain.com`, `service2.yourdomain.com` 
and any other subdomains without needing separate entries for each.

## Which one to use

If you are just getting started use MagicDNS — it requires no configuration 
beyond enabling Allow local network access in the Tailscale app and updating 
your bookmarks to use the hostname instead of the LAN IP.

Split DNS is worth setting up if you have services behind a reverse proxy with 
custom domain names and want them to route directly through Tailscale rather 
than resolving publicly.

Both can be used at the same time — MagicDNS for direct server access and 
Split DNS for domain based services.
