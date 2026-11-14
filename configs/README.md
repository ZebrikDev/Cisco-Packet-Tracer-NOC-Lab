# Router configuration reference

The settings below summarize the router interfaces, DHCP pools, and static routes shown in the captured running configurations.

## Interface assignments

| Router | LAN-facing interface | Transit interface |
|---|---|---|
| R1 | G0/0: `192.168.10.1 255.255.255.0` | G0/1: `10.0.0.1 255.255.255.252` |
| R2 | G0/0: `192.168.20.1 255.255.255.0` | G0/1: `10.0.0.2 255.255.255.252` |

Both interfaces on each router use `no shutdown`.

## R1 DHCP and static route

```text
ip dhcp excluded-address 192.168.10.1 192.168.10.99
ip dhcp excluded-address 192.168.10.200 192.168.10.254
ip dhcp pool LAN_A
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 exit
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

## R2 DHCP and static route

```text
ip dhcp excluded-address 192.168.20.1 192.168.20.99
ip dhcp excluded-address 192.168.20.200 192.168.20.254
ip dhcp pool LAN_B
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 exit
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

## Captured running configurations

| Router | Transcription | Original capture |
|---|---|---|
| R1 | [R1-running-config.txt](R1-running-config.txt) | [R1 CLI screenshot](../screenshots/r1-running-config.png) |
| R2 | [R2-running-config.txt](R2-running-config.txt) | [R2 CLI screenshot](../screenshots/r2-running-config.png) |

The captures show the interfaces, DHCP pools, and static routes. Both end after `line aux 0`, so the transcriptions cover the visible excerpt. Repeated separators were condensed and the simulated device identifier was omitted from the text.
