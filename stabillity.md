# Making it Stable

Everything might work perfectly the first time you bring the stack up and then 
mysteriously break after a restart. This section covers the two issues that 
cause that and how to fix them permanently.

## The Gluetun iptables problem

Gluetun adds DROP rules to the host's Docker iptables chain every time it 
starts. This is intentional — it's Gluetun's kill switch preventing traffic 
from leaking if the VPN drops. The problem is these rules are too broad and 
end up blocking Pi-hole from accepting DNS queries from devices outside its 
container network.

The symptom is DNS silently failing after any restart. Pi-hole appears healthy, 
the containers are all running, but queries time out. Running this will confirm 
it:

```bash
sudo iptables -L DOCKER -n | tail -10
```

If you see DROP rules at the bottom of the output that is the problem. The 
`iptables-fix` container in the exit node compose handles this automatically 
but only covers that stack. If you run other Gluetun containers on the same 
host — for example a separate download VPN — they will add their own DROP rules 
that the iptables-fix container does not cover.

Fix this by adding a cron job that clears DROP rules every five minutes 
regardless of which Gluetun container added them:

```bash
sudo crontab -e
```

Add this line:
*/5 * * * * for i in 1 2 3 4 5; do /sbin/iptables -D DOCKER -j DROP 2>/dev/null; done

This silently removes up to five DROP rules every five minutes and does nothing 
if there are none to remove. It does not affect Gluetun's internal kill switch 
which operates inside the container's own network namespace — your VPN leak 
protection stays intact.

If DNS breaks and you need it back immediately without waiting for the cron job 
run this manually:

```bash
for i in 1 2 3 4 5; do sudo /sbin/iptables -D DOCKER -j DROP 2>/dev/null; done
```

## The Pi-hole v6 listening mode problem

Pi-hole v6 changed how it handles DNS listening mode. Setting 
`DNSMASQ_LISTENING=all` in the compose environment is not enough — Pi-hole 
stores its config internally and the environment variable does not always 
override it on restart.

The symptom is Pi-hole logging warnings like this:
ignoring query from non-local network 100.x.x.x

Your Tailscale devices are reaching Pi-hole but Pi-hole is refusing to answer 
them because it considers their IP addresses outside its local network.

Fix it by setting the listening mode directly via the FTL config command:

```bash
docker exec -it pihole pihole-FTL --config dns.listeningMode all
```

This writes the setting to Pi-hole's internal config file where it survives 
restarts. Run it once after the initial setup and you should not need to run 
it again. Verify it stuck after a restart with:

```bash
docker exec -it pihole pihole-FTL --config dns.listeningMode
```

Should return `ALL`.

## Verifying everything is healthy after a restart

After any restart run through these quickly to confirm the stack is working:

```bash
# Check Pi-hole is resolving
dig @YOUR_SERVER_IP google.com

# Check Pi-hole is blocking
dig @YOUR_SERVER_IP doubleclick.net +short

# Check no DROP rules are present
sudo iptables -L DOCKER -n | grep DROP

# Check Gluetun is connected to VPN
docker logs gluetun-exit | grep "public IP"

# Check Tailscale exit node is running
docker exec -it tailscale-exit tailscale status
```

If `dig @YOUR_SERVER_IP` times out and DROP rules are present, flush them 
manually and DNS will come back immediately.

With the stack stable the next step is setting up blocklists to maximize what 
Pi-hole filters.
