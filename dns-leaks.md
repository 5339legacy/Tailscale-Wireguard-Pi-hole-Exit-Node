# Blocking DNS Leaks

Pi-hole and the browser settings cover most DNS leakage but there is one 
remaining vector — DNS over TLS on port 853. Some apps and operating systems 
bypass system DNS entirely and connect directly to a DoT resolver like 
Cloudflare on port 853. Pi-hole cannot block these because they never ask 
Pi-hole anything — they go straight to the IP address.

The fix is blocking port 853 outbound at the iptables level on your server. 
Any device routing traffic through your exit node cannot use DoT because the 
tunnel itself blocks it.

## What this does and does not affect

Before applying this it is worth understanding the scope. These rules block 
port 853 on the FORWARD chain which affects traffic passing through your 
server via the Tailscale exit node. It does not affect your server's own 
outbound traffic so Unbound continues to work normally.

If you run other Gluetun containers on the same host for downloading or other 
purposes those containers use their own network namespace and are not affected 
by these rules.

## Apply the rules

```bash
sudo iptables -I FORWARD -i tailscale0 -p tcp --dport 853 -j REJECT
sudo iptables -I FORWARD -i tailscale0 -p udp --dport 853 -j REJECT
```

Save them so they survive a reboot:

```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

When prompted whether to save current IPv4 and IPv6 rules select yes for both.

Verify the rules are in place:

```bash
sudo iptables -L FORWARD -n -v | grep 853
```

Should show two REJECT rules scoped to the tailscale0 interface.

## Verify the leak is closed

Run a DNS leak test at `https://dnsleaktest.com` from a device using your 
exit node. With DoT blocked and Pi-hole handling DNS you should see one 
unknown server in the results rather than Cloudflare or your ISP. That unknown 
server is your Unbound instance doing pure recursive resolution — it has no 
public identity which is exactly what you want.

If you still see Cloudflare in the results check that your browser DoH is 
disabled as covered in the browser settings section. Browser level DoH uses 
port 443 which cannot be blocked without breaking normal HTTPS traffic — the 
browser settings are the only way to close that gap.

[Split DNS for self hosted services →](split-dns.md)
