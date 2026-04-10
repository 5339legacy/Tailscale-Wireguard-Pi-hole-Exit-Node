# Tailscale DNS Configuration

With Pi-hole and the exit node running the last piece is telling Tailscale to 
use Pi-hole for DNS. Without this your devices will still resolve through 
whatever DNS they were using before — usually Cloudflare or your ISP.

Everything in this section happens in the Tailscale admin console at 
`https://login.tailscale.com/admin/dns`. You do not need to touch your server.

Go to the DNS page in the admin console. Under Global nameservers click 
Add nameserver and select Custom. Enter the Tailscale IP of your server — 
this is the `100.x.x.x` address assigned to your server by Tailscale, not 
its LAN IP. You can find it at `https://login.tailscale.com/admin/machines`. 
Check Use with exit node and save.

Make sure Override DNS servers is toggled on. This is what forces all devices 
on your tailnet to use Pi-hole instead of their own DNS settings. Without it 
devices will use Pi-hole for Tailscale hostnames but fall back to their own 
DNS for everything else.

Your Tailscale IP is the correct address to use here rather than your server's 
LAN IP. When a device is using the exit node it routes all traffic through the 
VPN tunnel — it cannot reach your LAN IP directly. The Tailscale IP is always 
reachable regardless of where the device is or whether the exit node is active.

Verify it is working by checking that your devices resolve DNS through Pi-hole. 
On any Tailscale device run:

```bash
nslookup doubleclick.net
```

If it returns `0.0.0.0` Pi-hole is handling DNS and blocking ads. If it 
returns a real IP address something is wrong with the nameserver config.

You can also open the Pi-hole dashboard at `http://YOUR_SERVER_TAILSCALE_IP:8080/admin` 
from any device on your tailnet and watch queries come in from your devices in 
real time.

One thing worth knowing — when you run a DNS leak test you may see your home 
IP appearing as the DNS resolver rather than your VPN provider's IP. This is 
expected. Unbound does pure recursive resolution directly to root servers, 
which means queries originate from your server's IP. No single DNS provider 
sees all your queries — they are split across hundreds of authoritative servers. 
This is more private than forwarding everything to Cloudflare even though it 
looks different on a leak test.

With DNS configured your devices are now routing traffic through the VPN and 
resolving DNS through Pi-hole. The next step is making sure this stays stable 
across restarts.
