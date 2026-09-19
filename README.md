## Topology Overview
 [PC0]--\                                                         /--[PC2]
         [Switch0]--Router0 ==== Router1 ==== Router2 ==== Router3--[Switch1]
 [PC1]--/   (Area 1)   (ABR)    (Area 0)     (Area 0)  (ABR)   (Area 2)  \--[PC3]
```

| Area   | Role                          | Networks                          |
|--------|-------------------------------|-----------------------------------|
| Area 1 | Left LAN                      | 10.10.10.0                        |
| Area 0 | Backbone (Router1 / links)    | 20.20.20.0, 30.30.30.0            |
| Area 2 | Right LAN                     | 40.40.40.0 (link), 50.50.50.0     |

> Router0 connects Area 1 to Area 0, and Router2 connects Area 0 to the Area 2 side, so both act as Area Border Routers (ABRs). Router1 is a backbone (Area 0) internal router.

## Devices

- 4 × Cisco ISR4331 routers (Router0, Router1, Router2, Router3)
- 2 × Cisco 2960-24TT switches (Switch0, Switch1)
- 4 × PCs (PC0, PC1, PC2, PC3)

## IP Addressing Table

| Device   | Interface | IP Address  | Connected To            |
|----------|-----------|-------------|-------------------------|
| Router0  | Gig0/0/0  | 10.10.10.1  | Switch0 (Fa0/1)         |
| Router0  | Se0/1/0   | 20.20.20.1  | Router1 (Se0/1/0)       |
| Router1  | Se0/1/0   | 20.20.20.2  | Router0 (Se0/1/0)       |
| Router1  | Se0/1/1   | 30.30.30.1  | Router2 (Se0/1/0)       |
| Router2  | Se0/1/0   | 30.30.30.2  | Router1 (Se0/1/1)       |
| Router2  | Se0/1/1   | 40.40.40.1  | Router3 (Se0/1/0)       |
| Router3  | Se0/1/0   | 40.40.40.2  | Router2 (Se0/1/1)       |
| Router3  | Gig0/0/0  | 50.50.50.1  | Switch1 (Fa0/1)         |
| PC0      | Fa0       | 10.10.10.2  | Switch0 (Fa0/2)         |
| PC1      | Fa0       | 10.10.10.3  | Switch0 (Fa0/3)         |
| PC2      | Fa0       | 50.50.50.2  | Switch1 (Fa0/2)         |
| PC3      | Fa0       | 50.50.50.3  | Switch1 (Fa0/3)         |

**Default gateways:** PC0 / PC1 → `10.10.10.1`, PC2 / PC3 → `50.50.50.1`

## Example OSPF Configuration

Assuming /24 subnets (`255.255.255.0`, wildcard `0.0.0.255`).

**Router0 (ABR: Area 1 / Area 0)**
```
router ospf 1
 network 10.10.10.0 0.0.0.255 area 1
 network 20.20.20.0 0.0.0.255 area 0

**Router1 (Backbone)**
```
router ospf 1
 network 20.20.20.0 0.0.0.255 area 0
 network 30.30.30.0 0.0.0.255 area 0

**Router2 (ABR: Area 0 / Area 2)**
router ospf 1
 network 30.30.30.0 0.0.0.255 area 0
 network 40.40.40.0 0.0.0.255 area 2

**Router3 (Area 2)**
router ospf 1
 network 40.40.40.0 0.0.0.255 area 2
 network 50.50.50.0 0.0.0.255 area 2

> Note: On the serial links, the DCE side (clock icon in Packet Tracer) needs a clock rate, e.g. `clock rate 64000`.

## Verification Commands
show ip route
show ip route ospf
show ip ospf neighbor
show ip protocols
show ip ospf interface brief

Connectivity test: ping from **PC0 (10.10.10.2)** to **PC2 (50.50.50.2)** and **PC3 (50.50.50.3)**. Replies confirm that routes are exchanged across all three areas. Routes learned from other areas appear as `O IA` (inter-area) in the routing table.

## How to Use

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer).
2. Clone or download this repository.
3. Open the `.pkt` file in Packet Tracer.
4. Check the configuration on each router and run the verification commands above.

## Concepts Covered

- Multi-area OSPF design (Area 0 backbone + Area 1 and Area 2)
- Area Border Routers (ABRs) and inter-area routes
- Serial WAN links with DCE/DTE clocking
- LAN setup with switches, PCs and default gateways
- OSPF neighbor verification and end-to-end testing
