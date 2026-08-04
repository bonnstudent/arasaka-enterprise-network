# pfSense IPSec — Self-Originated Pings Fail Through Tunnels

**Environment:** pfSense firewalls at three sites, site-to-site IPSec tunnels (IKEv2)

## Symptom

After bringing up the site-to-site IPSec tunnels, pings issued from the pfSense shell to
hosts at remote sites failed. This suggested the tunnels were not working, even though
the Phase 1 and Phase 2 entries showed as established.

## What I checked

1. **Tunnel status** — Phase 1 and Phase 2 both showed established in the IPSec status page.
2. **Phase 2 selectors** — local and remote networks were defined correctly for each subnet.
3. **Firewall rules** — the IPSec interface permitted the traffic.
4. **Client-to-client traffic** — pings between actual hosts at different sites succeeded.

That last check was the key one. Real client traffic crossed the tunnels without issue,
which meant the tunnels were fine and the problem was specific to traffic originating
from pfSense itself.

## Root cause

Traffic originating from the pfSense host does not automatically select a source address
inside the tunnel's Phase 2 local network. Without a matching source address, the packet
does not match the IPSec security policy and is not encapsulated, so it never enters the
tunnel.

## Fix

Specify the source address explicitly when testing from the pfSense shell:

    ping -S <local LAN IP> <remote host>

With a source address inside the Phase 2 local network, the traffic matches the policy
and traverses the tunnel normally.

## Validation

- Ping with `-S` succeeded to hosts at both remote sites
- Client-to-client traffic across sites continued to work unaffected
- Confirmed the behaviour was diagnostic-only and not a topology fault

## Takeaway

A failing test does not always mean a failing system. The tool used to test can have
behaviour that differs from the traffic it is meant to simulate — worth confirming with
a real client before concluding the infrastructure is broken.
