
# Tailscale WireGuard Pi-hole Exit Node

This is a guide for building a privacy focused network stack that gives you 
encrypted VPN coverage, network wide ad blocking, and private DNS on every 
device you own — at home or anywhere in the world.

When it's done your devices will permanently route through a WireGuard VPN exit 
node with Pi-hole blocking ads and trackers at the DNS level. No Cloudflare. 
No Google. No single company seeing your DNS queries. Just your infrastructure 
doing exactly what you tell it to.

## What you end up with

- All traffic on your devices exits through a WireGuard VPN
- DNS handled by Pi-hole with custom blocklists
- Unbound doing pure recursive resolution — no upstream DNS provider
- Tailscale connecting everything securely from anywhere
- Self hosted services reachable via Split DNS regardless of where you are
- No DNS leaks via DoT blocking

## What you need

- A Linux server running Docker — this guide was built and tested on Ubuntu 24
  with Docker 27 and Docker Compose v2
- A consistent LAN IP for your server — if your server has been running for a
  while your router has almost certainly given it the same address every time.
  If you want to guarantee it never changes set a DHCP reservation in your
  router settings. Either way confirm the IP before starting and make note of
  it — you will use it throughout this guide
- Port 53 available on the host — if systemd-resolved is running you will need
  to disable it first. Check with `systemctl status systemd-resolved`
- A Tailscale account — free tier is fine
- A WireGuard VPN provider — NordVPN, Mullvad, ProtonVPN, anything that gives
  you WireGuard credentials works. You will need your private key and a server
  endpoint. Check your provider's documentation for how to generate these
- A domain name — only needed for the Split DNS section, you can skip that
  section if you don't have one

## How it fits together
Your devices
└── Tailscale
└── Gluetun (WireGuard VPN)
└── Traffic exits through your VPN provider
└── DNS → Pi-hole → Unbound → root servers

## Performance

Being honest about this — there is a latency overhead. On a home gigabit 
connection the speed difference is barely noticeable. On cellular you will 
feel around 20ms of added delay when sites are resolving, sometimes more 
depending on your signal and location.

Once a page starts loading the speeds themselves are fine. It's the initial 
DNS resolution and connection handshake where you notice it most — that 
slight pause before a site begins to load.

Whether that trade-off is worth it is up to you. For most people who care 
about privacy it becomes background noise within a day or two. You stop 
noticing it the same way you stop noticing a seatbelt.

Raw numbers from this setup:
- Home connection without VPN: ~11ms
- Through the full stack: ~25ms
- Download speeds: near gigabit
- Upload: limited by your home upload speed

## What this doesn't protect you from

This stack protects you at the network level. It does not make you anonymous.

- If you are logged into Google, Google still knows what you are doing
- Browser fingerprinting can still identify your device regardless of VPN
- Browsers with DNS over HTTPS enabled will bypass Pi-hole — disable DoH in
  your browser settings
- Your VPN provider can see your traffic — choose one you trust with a
  credible no logs policy

## Honest assessment of the build process

This took a full day to get right. The hardest parts were:

- Gluetun's firewall adding DROP rules to the host iptables that silently
  break Pi-hole every time it restarts — there is a fix for this in the
  stability section and it is not obvious
- Pi-hole v6 changed how DNS listening mode works — the environment variable
  alone is not enough, you have to set it via the FTL config command
- Getting Tailscale DNS to actually route through Pi-hole rather than
  Cloudflare took more debugging than expected

None of it is insurmountable but go in knowing it is not a straight line.
The stability section covers every gotcha we hit.

# Start here
1. [Pi-hole and Unbound](pihole-unbound.md)
2. [Gluetun WireGuard Exit Node](exit-node.md)
3. [Tailscale DNS Configuration](tailscale-dns.md)
4. [Making it Stable](stability.md)
5. [Blocklists](blocklists.md)
6. [Browser Settings](browser-settings.md)
7. [Blocking DNS Leaks](dns-leaks.md)
8. [Accessing Self Hosted Services Remotely](local-access.md)

## Contributing

If something in this guide is wrong, outdated, or could be clearer open an 
issue. This was built through a lot of trial and error and there are probably 
better ways to do parts of it.

## Why I built this

Your ISP sells your browsing data. Public DNS providers like Cloudflare and 
Google log your queries. Every device you own is constantly phoning home to 
someone. This stack cuts most of that off at the network level without 
requiring you to change how you use the internet day to day.

The performance impact is minimal — WireGuard is fast enough that you likely 
won't notice it's running.
