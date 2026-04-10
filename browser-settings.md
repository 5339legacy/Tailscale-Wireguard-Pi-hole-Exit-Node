# Browser Settings

The network stack handles DNS and VPN at the infrastructure level but browsers 
have their own DNS settings that can bypass everything you just built. This 
section covers locking down Brave specifically — it is the recommended browser 
for this setup but the concepts apply to any browser.

## Disable DNS over HTTPS in Brave

Brave has DNS over HTTPS enabled by default. When active it sends DNS queries 
directly to Cloudflare or another provider over HTTPS, completely bypassing 
Pi-hole regardless of your network configuration. Turn it off:

Go to `brave://settings/security` and find Use secure DNS. Turn it off 
completely. Do not set it to a custom provider — turn it off entirely so DNS 
falls through to the system resolver which routes through Pi-hole.

Disabling DNS over HTTPS in the browser is not a security risk in this setup. 
DoH was designed to protect DNS queries from being snooped on by your ISP or 
anyone on your network. Your setup already solves that problem — all traffic 
routes through a WireGuard VPN tunnel which encrypts everything including DNS. 
You are trading browser level DNS encryption for network level VPN encryption 
which is a stronger guarantee. The only thing you lose is Cloudflare handling 
your DNS which is exactly what you are trying to avoid.

## Disable DNS over HTTPS at the OS level on Windows

Windows 11 has its own DoH setting separate from the browser. Even with Brave's 
DoH off, Windows can still send encrypted DNS queries that bypass Pi-hole. 
Open PowerShell as administrator and run:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters" -Name "EnableAutoDoh" -Value 0
```

Reboot for the change to take effect. Verify it worked:

```powershell
netsh dns show global
```

Should show DoH settings as disabled.

## Enable Brave Shields

Brave's built in shields block ads, trackers and fingerprinting at the browser 
level. This works alongside Pi-hole — Pi-hole blocks at the DNS level before 
the request leaves your device, shields block at the page level after it loads. 
Together they cover more than either does alone.

Click the Brave lion icon on any page and set:
- Trackers and ads blocked → Aggressive
- Fingerprinting blocked → Strict

## Clear cookies on close

Cookies are how sites track you across sessions. Setting Brave to clear them 
automatically on close means every session starts fresh with no persistent 
tracking state.

Go to `brave://settings/privacy` and enable Clear cookies and site data when 
you close all windows.

You will be logged out of everything each time you close the browser. That 
friction is intentional — it prevents passive tracking across sessions and 
forces you to be deliberate about which sites get persistent access.

## Search engine

Go to `brave://settings/search` and set your default search engine to 
Brave Search. Unlike Google or Bing, Brave Search has its own independent 
index and does not track your queries or build a profile on you.

DuckDuckGo is a reasonable alternative but it partially relies on Bing's index 
which means Microsoft sees your queries. Brave Search is the cleaner option.

## What browser settings cannot protect you from

Changing browser settings reduces passive tracking significantly but does not 
make you anonymous. If you are logged into any account — Google, Facebook, 
Amazon — that service knows what you are doing on their platform regardless 
of your DNS setup or browser shields. The protections here are most effective 
when browsing without being logged in.

[Blocking DNS leaks →](dns-leaks.md)
