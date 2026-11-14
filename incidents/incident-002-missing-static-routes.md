# Incident 002: Missing Static Routes Between LANs

**Project:** Cisco Packet Tracer – NOC Network Configuration & Troubleshooting Lab  
**Environment:** Packet Tracer simulation lab  
**Status:** Resolved. PC1 reached the LAN B gateway again, and both LAN B clients obtained DHCP addresses and reached their local gateway.  
**Duration:** Not measured.

## Symptom and scope

PC1 could not ping the LAN B gateway at `192.168.20.1`. During the exercise, the transit link between R1 and R2 responded to ping. After PC1 recovered a valid DHCP configuration and could ping its local gateway, that gateway returned ICMP Destination Host Unreachable for the remote test. The diagnosed scope was the routed path between the LANs.

## Controlled change and diagnosis

I simulated a routing failure by removing R1's static route to `192.168.20.0/24` and verified the change in R1's routing table. After I restored the route, PC1 still timed out when pinging the LAN B gateway. I checked R2 and found that its return route to `192.168.10.0/24` was also missing. The timing and cause of the missing return route remained undetermined.

| Check | Finding | Source screenshot |
|---|---|---|
| R1 ping to 10.0.0.2 | 5/5 replies | [incident-002-transit-ping](../screenshots/incident-002-transit-ping.png) |
| PC1 IPv4 configuration and local gateway ping | 192.168.10.100/24, gateway 192.168.10.1, 4/4 replies | [incident-002-local-gateway](../screenshots/incident-002-local-gateway.png) |
| PC1 ping to 192.168.20.1 | Unreachable messages from 192.168.10.1 and a timeout | [incident-002-unreachable](../screenshots/incident-002-unreachable.png) |
| R1 routing table with fault active | No route to 192.168.20.0/24 | [incident-002-r1-route-missing](../screenshots/incident-002-r1-route-missing.png) |
| R2 routing table before repair | No route to 192.168.10.0/24 | [incident-002-r2-route-missing](../screenshots/incident-002-r2-route-missing.png) |

## Corrective action

R1 needed a route for the request and R2 needed a route for the reply.

| Router | Restored command |
|---|---|
| R1 | `ip route 192.168.20.0 255.255.255.0 10.0.0.2` |
| R2 | `ip route 192.168.10.0 255.255.255.0 10.0.0.1` |

When I reopened the saved project, both static routes were missing, so I restored them again. PC1 also lacked a usable IPv4 configuration. I recreated the LAN A DHCP pool on R1 and renewed PC1's lease before testing the routed path. On LAN B, PC3 and PC4 initially lacked DHCP addresses. After recovering DHCP, I verified that both clients obtained leases and could reach their local gateway.

## Recovery verification

| Check | Result | Evidence |
|---|---|---|
| R1 routing table after initial restoration | Static route to 192.168.20.0/24 via 10.0.0.2 | [incident-002-r1-route-restored](../screenshots/incident-002-r1-route-restored.png) |
| R2 routing table after final restoration | Static route to 192.168.10.0/24 via 10.0.0.1 | [incident-002-r2-route-restored](../screenshots/incident-002-r2-route-restored.png) |
| PC1 ping to 192.168.20.1 after restoration | First request timed out, next three received replies | [incident-002-gateway-ping-restored](../screenshots/incident-002-gateway-ping-restored.png) |
| PC1 repeat ping to 192.168.20.1 | Gateway reachable | Lab observation |
| PC3 and PC4 DHCP and gateway connectivity | Both obtained leases and reached their gateway | Lab observation |

The cause of the first post-repair timeout was not verified. Later end-to-end verification showed [4/4 replies from PC1 to the LAN B host](../screenshots/cross-lan-ping.png). The [final validation capture](../screenshots/final-cross-lan-ping.png) shows 2/4 replies followed by 4/4 on a repeat. These captures verify the tested path.

## Takeaways

Check the source host's address and local gateway before diagnosing the remote path. An unreachable reply from the gateway identifies a device that the host can reach and narrows the routing investigation. Check both routing tables, and save the router startup configurations and the Packet Tracer project after recovery.
