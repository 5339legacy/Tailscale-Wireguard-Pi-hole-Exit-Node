markdown# Blocklists

Pi-hole ships with a basic default blocklist. These are the lists worth adding 
on top of it. Some are aggressive and will occasionally block legitimate 
services — when something stops working check the Pi-hole query log first 
before assuming it is a different problem. Nine times out of ten a broken 
service is a blocked domain.

Add lists in the Pi-hole dashboard under Lists. After adding all of them run 
gravity update:

```bash
docker exec -it pihole pihole updateGravity
```

## General ad and tracking blocking
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/multi.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.plus.txt
https://big.oisd.nl/
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts

## Device telemetry

These block native tracking from specific manufacturers. Only add the ones 
relevant to devices you actually own.
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.samsung.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.apple.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.winoffice.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.android.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.lgwebos.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.roku.txt

## Smart TV
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt

## Security and threat intelligence
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt
https://phishing.army/download/phishing_army_blocklist_extended.txt

## DoH bypass prevention

Blocks known DNS over HTTPS resolver domains so browsers and apps cannot 
bypass Pi-hole using encrypted DNS. This works at the domain level — pair it 
with the iptables rules in the DNS leaks section for complete coverage.
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/doh.txt

## Mobile tracking
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/android-tracking.txt

With a full set of lists running you should expect somewhere between 20-40% 
of DNS queries to be blocked depending on the devices on your network and 
your browsing habits. Higher is not necessarily better — it just means more 
telemetry was being sent before you blocked it.
