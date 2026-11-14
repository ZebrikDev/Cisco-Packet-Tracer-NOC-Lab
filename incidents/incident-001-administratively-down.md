# Incident 001: Administratively Disabled LAN Interface

**Project:** Cisco Packet Tracer – NOC Network Configuration & Troubleshooting Lab  
**Environment:** Simulation lab, Cisco Packet Tracer 9.0.0.0810  
**Status:** Resolved. The recovery captures show the interface up and the remote host reachable.  
**Incident and recovery times:** Not recorded. No response-time or recovery-duration metric is claimed.

## Symptom

I checked PC1's IPv4 settings and found them intact. A ping to its default gateway, `192.168.10.1`, and a subsequent `tracert` to `192.168.20.1` both timed out.

## Scope

- **Observed impact:** PC1 could not reach its default gateway.
- **Unaffected path verified by screenshot:** R2 reached R1 at 10.0.0.1, with 5/5 successful ping replies.
- **Inferred impact:** Routed traffic using R1's LAN A interface was unavailable while that interface was disabled.
- PC2's failure result and PC1-to-PC2 switching were not captured during the incident. Their outcomes are not asserted here.

## Initial hypotheses

I considered three possible fault locations: PC1's access path, the LAN A link to its gateway, and the gateway interface. A successful ping from R2 to R1 confirmed that the transit path still worked, so I checked R1's LAN-facing interface.

## Investigation and evidence

| Check | Result | Evidence |
|---|---|---|
| PC1: ipconfig | IPv4 settings were intact | Lab observation |
| PC1: ping 192.168.10.1 | Timed out | Lab observation |
| PC1: tracert 192.168.20.1 | Timed out | Lab observation |
| R2: ping 10.0.0.1 | 100% success, 5/5 replies | [incident-001-transit-ping](../screenshots/incident-001-transit-ping.png) |
| R1: show ip interface brief | G0/0 administratively down/down; G0/1 up/up | [incident-001-interface-down](../screenshots/incident-001-interface-down.png) |

The interface table showed:

| R1 interface | IPv4 address | Status | Protocol |
|---|---|---|---|
| GigabitEthernet0/0 | 192.168.10.1 | administratively down | down |
| GigabitEthernet0/1 | 10.0.0.1 | up | up |

## Root cause

R1 GigabitEthernet0/0, the LAN A gateway interface, had been administratively disabled. The transit interface remained operational, explaining why R2 could still reach R1.

### Controlled fault injection

To simulate the outage, I disabled R1's LAN-facing interface in a working copy of the lab.

```text
configure terminal
interface GigabitEthernet0/0
shutdown
end
```

I kept a separate copy of the working baseline.

## Corrective action and recovery verification

I re-enabled the interface with `no shutdown`.

```text
configure terminal
interface GigabitEthernet0/0
no shutdown
end
```

I checked that G0/0 returned to `up/up`, then tested reachability from PC1 to its gateway and the remote host.

| Recovery check | Verified result | Evidence |
|---|---|---|
| R1: show ip interface brief | G0/0 at 192.168.10.1 and G0/1 at 10.0.0.1 both up/up | [incident-001-interface-recovered](../screenshots/incident-001-interface-recovered.png) |
| R1: copy running-config startup-config | Completed with [OK] | [incident-001-interface-recovered](../screenshots/incident-001-interface-recovered.png) |
| PC1: ping 192.168.10.1 | Gateway reachable | Lab observation |
| PC1: ping 192.168.20.101 | 4 sent, 4 received, 0 lost (0% loss) | [incident-001-ping-recovered](../screenshots/incident-001-ping-recovered.png) |

## Learning outcomes

- Correct IPv4 settings do not establish gateway reachability.
- Reachability to one router interface does not establish reachability to every interface.
- Administratively down identifies an interface disabled through configuration.
- enable changes the CLI privilege level; no shutdown enables the selected interface.
- Record observed scope separately from inferred impact.
