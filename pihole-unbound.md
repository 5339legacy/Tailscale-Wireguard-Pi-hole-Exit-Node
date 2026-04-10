# Pi-hole and Unbound

This is the foundation of the whole stack. Pi-hole handles DNS filtering and 
ad blocking. Unbound sits behind it doing pure recursive resolution — meaning 
your DNS queries go directly to the root servers without passing through 
Cloudflare, Google, or any other third party.

## How they work together

Most Pi-hole setups forward DNS upstream to Cloudflare or Google. That defeats 
a lot of the privacy benefit — you're blocking ads but still telling Cloudflare 
every domain you visit. Unbound fixes that by resolving domains recursively 
from the root, splitting your queries across hundreds of authoritative servers 
instead of one company seeing everything.

The trade-off is a slight increase in resolution time on the first query for 
a domain. After that it's cached and fast.

## The setup

Both run as Docker containers on the same bridge network so they can talk to 
each other directly.

Create a directory for this stack:

```bash
mkdir ~/docker/DNS
cd ~/docker/DNS
```

Create the compose file:

```bash
nano docker-compose.yml
```

Paste this:

```yaml
services:
  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    restart: unless-stopped
    networks:
      dns_net:
        ipv4_address: 172.25.0.2

  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8080:80/tcp"
    environment:
      TZ: "America/Los_Angeles"
      FTLCONF_dns_upstreams: "172.25.0.2#53"
      FTLCONF_webserver_api_password: "changeme"
      DNSSEC: "true"
      DNSMASQ_LISTENING: "all"
      FTLCONF_dns_listeningMode: "all"
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    networks:
      dns_net:
        ipv4_address: 172.25.0.3
    depends_on:
      - unbound

networks:
  dns_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/24
```

Change `America/Los_Angeles` to your timezone and set a real password for 
`FTLCONF_webserver_api_password`.

Bring it up:

```bash
docker compose up -d
```

## Verify it's working

Test Pi-hole is resolving:

```bash
dig @YOUR_SERVER_IP google.com
```

Test Pi-hole is blocking ads:

```bash
dig @YOUR_SERVER_IP doubleclick.net +short
```

Should return `0.0.0.0` if blocking is working.

Test Pi-hole can reach Unbound:

```bash
docker exec -it pihole nslookup google.com 172.25.0.2
```

## One thing Pi-hole v6 changed

Pi-hole v6 stores its config differently than v5. The `DNSMASQ_LISTENING` 
environment variable alone is not enough to make Pi-hole accept queries from 
outside its local network — which you need for Tailscale devices to reach it.

After the container is running, set the listening mode directly:

```bash
docker exec -it pihole pihole-FTL --config dns.listeningMode all
```

This writes the setting to Pi-hole's internal config and survives restarts. 
Without this your Tailscale devices will get `ignoring query from non-local 
network` errors and DNS will silently fail.

## Access the web UI

Pi-hole's dashboard is at `http://YOUR_SERVER_IP:8080/admin`

From here you can see query logs, manage blocklists, and monitor what's being 
blocked across your network.

## Next

With Pi-hole and Unbound running, the next step is setting up the WireGuard 
exit node that routes your device traffic through your VPN provider.

[Gluetun WireGuard Exit Node →](exit-node.md)
