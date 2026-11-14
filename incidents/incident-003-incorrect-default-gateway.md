# Incident 003: Incorrect Default Gateway on PC1

**Project:** Cisco Packet Tracer – NOC Network Configuration & Troubleshooting Lab  
**Environment:** Packet Tracer simulation lab  
**Status:** Resolved. PC1 reached the remote host after its default gateway was corrected.  
**Duration:** Not measured.

## Baseline and controlled change

In the incident copy, I set PC1's static IPv4 address to `192.168.10.50/24` and its default gateway to `192.168.10.1`. I verified 4/4 replies from both the local gateway and the LAN B host at `192.168.20.100`. To simulate the fault, I changed only PC1's default gateway to `192.168.10.254`.

## Symptom and scope

PC1 could still reach `192.168.10.1` (4/4 replies). All four probes to the LAN B host timed out. PC2 reached that remote host in a separate test (2/4 replies after two initial timeouts), showing the remote path was still reachable from another LAN A workstation. The two PC2 timeouts were observed but their cause was not established.

## Investigation

| Stage | Test or evidence | Result | Original screenshot |
|---|---|---|---|
| Before injection | PC1 ipconfig and pings | Gateway 192.168.10.1; both targets 4/4 | [incident-003-baseline](../screenshots/incident-003-baseline.png) |
| Fault active | PC1 ipconfig and pings | Gateway 192.168.10.254; local target 4/4; remote host 0/4 | [incident-003-wrong-gateway](../screenshots/incident-003-wrong-gateway.png) |
| Scope check | PC2 ping to 192.168.20.100 | 2/4 replies | [incident-003-pc2-scope-check](../screenshots/incident-003-pc2-scope-check.png) |
| After repair | PC1 ipconfig and ping to 192.168.20.100 | Gateway restored to 192.168.10.1; 4/4 replies, 0% loss | [incident-003-recovered](../screenshots/incident-003-recovered.png) |

The local ping remained successful because the target was in PC1's own subnet. The remote target required a usable gateway. The unchanged host address and mask, the incorrect gateway shown by `ipconfig`, and the comparison with PC2 supported a host configuration diagnosis.

## Root cause and recovery

PC1 was configured to use `192.168.10.254` as its next hop for off-subnet traffic. In PC1's **Desktop → IP Configuration**, I restored the gateway to R1's LAN A address, `192.168.10.1`, and kept PC1's static address `192.168.10.50/24` for the recovery test. PC1 received all four replies from the LAN B host, as shown in the recovery capture.

## Learning outcomes

Comparing local and remote reachability isolated the fault to traffic leaving PC1's subnet. Testing from PC2 helped me narrow the issue to PC1's configuration before making changes.
