# Subnet Failure Caused by Hypervisor NIC Mapping Mismatch

**Environment:** VMware Workstation, Cisco IOS CSR router, pfSense, Windows/Linux clients

## Symptom

One LAN subnet behind the Site 1 router passed no traffic. Clients on that segment could not reach their default gateway or anything beyond it. Other interfaces on the same router worked normally.

## What I checked

1. **Router interface configuration** — IP address, subnet mask, and no-shutdown state were all correct.
2. **Firewall rules** — nothing was blocking the segment.
3. **VM network settings** — the adapter appeared assigned to the correct LAN segment in VMware.
4. **Client configuration** — addressing and gateway were correct.

Every layer of configuration read as correct, which meant the fault was somewhere the configuration didn't describe.

## Diagnosis

Rather than continuing to re-read configuration files, I moved to testing the physical mapping directly.

I disconnected the virtual network adapters one at a time and observed interface state on the router after each change:

    show ip int br

By watching which interface changed to `down` as each adapter was disconnected, I could map each VMware adapter to the router interface it was actually bound to.

## Root cause

The VMware adapter labelled as one LAN segment was actually bound to a different router interface at the hypervisor level. The label in VMware Settings did not reflect the real underlying mapping.

## Fix

Swapped the LAN segment assignments between the two adapters and power-cycled the VM to force the mapping to reload.

## Validation

- Pinged from a client on the affected segment to its gateway
- Pinged across the IPSec tunnels to the other sites
- Confirmed domain authentication against the domain controller

## Takeaway

The layer I trusted most — the adapter labels — was the one that was wrong. When every configuration layer checks out, the assumption itself is worth testing. I now verify adapter-to-interface mappings rather than trusting labels.
