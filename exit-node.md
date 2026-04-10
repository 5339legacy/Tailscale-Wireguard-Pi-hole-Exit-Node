Then create the compose file:

```bash
nano docker-compose.yml
```

Paste this:

```yaml
services:
  gluetun-exit:
    image: qmcgaw/gluetun:latest
    container_name: gluetun-exit
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=Add your provider name here
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=${VPN_PRIVATE_KEY}
      - SERVER_COUNTRIES=Add your preferred country here
      - FIREWALL=on
      - FIREWALL_VPN_INPUT_PORTS=51820
      - FIREWALL_INPUT_PORTS=53
      - FIREWALL_OUTBOUND_SUBNETS=100.64.0.0/10,Add your LAN subnet here
      - VPN_LAN_NETWORK=100.64.0.0/10,Add your LAN subnet here
      - DOT=off
      - DNS_ADDRESSES=Add your server LAN IP here
      - DNS_KEEP_NAMESERVER=on
      - TZ=Add your timezone here
      - PUID=1000
      - PGID=1000
    volumes:
      - ./gluetun:/gluetun
    healthcheck:
      test: ["CMD-SHELL", "ping -c1 1.1.1.1 >/dev/null 2>&1 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s
    restart: unless-stopped

  tailscale-exit:
    image: tailscale/tailscale:latest
    container_name: tailscale-exit
    network_mode: "service:gluetun-exit"
    depends_on:
      gluetun-exit:
        condition: service_healthy
    cap_add:
      - NET_ADMIN
      - NET_RAW
    devices:
      - /dev/net/tun:/dev/net/tun
    volumes:
      - ./tailscale/state:/var/lib/tailscale
      - ./tailscale/config:/config
      - /dev/net/tun:/dev/net/tun
    environment:
      - TS_HOSTNAME=vpn-exit
      - TS_AUTHKEY=${TS_AUTHKEY}
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_EXTRA_ARGS=--advertise-exit-node --accept-dns=false --reset
      - TS_AUTH_ONCE=false
    restart: unless-stopped

  iptables-fix:
    image: alpine
    container_name: iptables-fix
    network_mode: host
    cap_add:
      - NET_ADMIN
    depends_on:
      gluetun-exit:
        condition: service_healthy
    command: >
      sh -c "iptables -D DOCKER -j DROP 2>/dev/null;
             iptables -D DOCKER -j DROP 2>/dev/null;
             echo 'DROP rules removed'"
    restart: on-failure
```

Replace `Add your provider name here` with your VPN provider name as Gluetun 
expects it — check the Gluetun docs at `https://github.com/qdm12/gluetun-wiki` 
for the exact string your provider uses. Replace `Add your LAN subnet here` 
with your home network subnet, usually `192.168.1.0/24` or similar — check 
with `ip route | grep default`. Replace `Add your server LAN IP here` with the 
LAN IP of your server where Pi-hole is running. Replace `Add your timezone here` 
with your timezone in tz format, for example `America/Los_Angeles`.

Bring everything up:

```bash
docker compose up -d
```

Verify Gluetun connected to your VPN provider:

```bash
docker logs gluetun-exit | grep -E "public IP|error"
```

You should see a public IP from your VPN provider. If you see errors check 
your WireGuard private key and provider name in the Gluetun wiki.

Tailscale requires you to manually approve exit nodes. Go to 
`https://login.tailscale.com/admin/machines`, find `vpn-exit` in the list, 
click the three dots, select Edit route settings, enable Use as exit node and 
save. Then verify Tailscale is running:

```bash
docker exec -it tailscale-exit tailscale status
```

You should see your other Tailscale devices listed and `vpn-exit` showing as 
offering an exit node.

Gluetun's firewall adds blanket DROP rules to the host's Docker iptables chain 
every time it starts. This is its kill switch — if the VPN drops, nothing 
leaks. The problem is these rules are too broad and block Pi-hole from 
accepting DNS queries from outside its container. The `iptables-fix` container 
runs once after Gluetun becomes healthy and removes those DROP rules 
automatically. Without it Pi-hole will randomly stop resolving DNS after any 
restart and the cause is not obvious.

With the exit node running the next step is configuring Tailscale DNS so your 
devices actually use Pi-hole for resolution.

[Tailscale DNS Configuration →](tailscale-dns.md)
