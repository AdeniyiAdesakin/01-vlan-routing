# Module 1: VLAN Segmentation and Inter-VLAN Routing

**Cisco 2960 | VLANs | Access Ports | 802.1Q | Router-on-a-Stick | DHCP | Wireless | Printers**

[← Return to the main project](../README.md)

## Module Overview

This module covers the campus foundation. I divided the network into departmental VLANs, assigned access ports, verified Layer 2 isolation, and then introduced router-on-a-stick so the VLANs could communicate through controlled Layer 3 gateways.

I also configured DHCP pools, added DNS options, connected wireless endpoints, and expanded the design with departmental printers. The walkthrough preserves the configuration evidence and identifies addressing or command-placement issues visible in the original lab.

## Objectives

- Create VLANs for Administration, Academics, Students, and Servers.
- Assign switch ports to the correct broadcast domains.
- Validate same-VLAN communication and pre-routing VLAN isolation.
- Configure an 802.1Q trunk and router subinterfaces.
- Provide dynamic addressing through Cisco IOS DHCP pools.
- Add wireless clients and departmental printers.
- Record successful tests and unresolved configuration issues accurately.

## Module Addressing

| VLAN | Department | Subnet | Gateway |
|---|---|---|---|
| `10` | Administration | `192.168.10.0/24` | `192.168.10.1` |
| `20` | Academics | `192.168.20.0/24` | `192.168.20.1` |
| `30` | Students | `192.168.30.0/24` | `192.168.30.1` |
| `100` | Server Room | `192.168.100.0/24` | `192.168.100.1` |

## Phase 1: Built the VLAN-Segmented Campus Network

### 1. Created and Named the Core Switch

I added a Cisco 2960 switch and renamed it **CoreSwitch** from the CLI.

```text
Switch> enable
Switch# configure terminal
Switch(config)# hostname CoreSwitch
```

<p align="center">
  <img src="https://i.imgur.com/i6at09t.png" width="750" alt="Selecting a Cisco 2960 switch in Packet Tracer">
</p>

<p align="center">
  <img src="https://i.imgur.com/hn3qMHT.png" width="750" alt="Renaming the switch CoreSwitch from the Cisco IOS CLI">
</p>

### 2. Created the Departmental VLANs

I created VLANs for Administration, Academics, Students, and the Server Room. The screenshot records earlier attempts to use spaces in VLAN names, which produced invalid-input messages. I then used underscore-separated names that Cisco IOS accepted.

```text
CoreSwitch(config)# vlan 10
CoreSwitch(config-vlan)# name Administration_Building
CoreSwitch(config-vlan)# vlan 20
CoreSwitch(config-vlan)# name Academics_Building
CoreSwitch(config-vlan)# vlan 30
CoreSwitch(config-vlan)# name Student_Building
CoreSwitch(config-vlan)# vlan 100
CoreSwitch(config-vlan)# name Server_Room
```

<p align="center">
  <img src="https://i.imgur.com/rmCLHj0.png" width="750" alt="Creating VLANs 10, 20, 30, and 100 and correcting VLAN names in Cisco IOS">
</p>

### 3. Assigned Access Ports to the VLANs

I assigned groups of FastEthernet ports to each VLAN and saved the configuration.

```text
CoreSwitch(config)# interface range fa0/2-5
CoreSwitch(config-if-range)# switchport access vlan 10
CoreSwitch(config)# interface range fa0/6-9
CoreSwitch(config-if-range)# switchport access vlan 20
CoreSwitch(config)# interface range fa0/10-14
CoreSwitch(config-if-range)# switchport access vlan 30
CoreSwitch(config)# interface range fa0/15-18
CoreSwitch(config-if-range)# switchport access vlan 100
CoreSwitch(config)# do write
```

<p align="center">
  <img src="https://i.imgur.com/fh0Q8YN.png" width="750" alt="Assigning FastEthernet port ranges to campus VLANs">
</p>

I then connected two endpoints to each VLAN and organized the topology by department.

<p align="center">
  <img src="https://i.imgur.com/eRX3ce5.png" width="750" alt="Initial campus topology with Administration, Academics, Student, and Server VLANs">
</p>

### 4. Validated VLAN Isolation

A ping between two hosts in VLAN 10 succeeded, confirming same-VLAN Layer 2 communication.

<p align="center">
  <img src="https://i.imgur.com/E9VeuoA.png" width="750" alt="Successful ping between two hosts in VLAN 10">
</p>

A ping from VLAN 10 to VLAN 20 failed before routing was configured, confirming that the VLANs were isolated.

<p align="center">
  <img src="https://i.imgur.com/cCcNbV1.png" width="750" alt="Failed cross-VLAN ping before inter-VLAN routing was configured">
</p>

## Phase 2: Configured Router-on-a-Stick Inter-VLAN Routing

### 1. Configured the 802.1Q Trunk

I converted CoreSwitch FastEthernet0/1 into a trunk so one physical link could carry traffic for multiple VLANs to the router.

```text
CoreSwitch(config)# interface fa0/1
CoreSwitch(config-if)# switchport mode trunk
CoreSwitch(config-if)# do write
```

<p align="center">
  <img src="https://i.imgur.com/llngkjD.png" width="750" alt="Configuring FastEthernet0/1 as an 802.1Q trunk">
</p>

### 2. Renamed and Enabled the Main-Campus Router Interface

I renamed the router **MC_router** and enabled GigabitEthernet0/1 as the parent interface for the VLAN subinterfaces.

<p align="center">
  <img src="https://i.imgur.com/y7XH0Yr.png" width="750" alt="Renaming the router MC_router">
</p>

```text
MC_router(config)# interface GigabitEthernet0/1
MC_router(config-if)# no shutdown
```

<p align="center">
  <img src="https://i.imgur.com/DIuLMtL.png" width="750" alt="Enabling the router GigabitEthernet0/1 interface">
</p>

### 3. Created VLAN Subinterfaces

I created one 802.1Q subinterface per VLAN and assigned each subinterface the default-gateway address for its subnet.

```text
MC_router(config)# interface GigabitEthernet0/1.10
MC_router(config-subif)# encapsulation dot1q 10
MC_router(config-subif)# ip address 192.168.10.1 255.255.255.0

MC_router(config)# interface GigabitEthernet0/1.20
MC_router(config-subif)# encapsulation dot1q 20
MC_router(config-subif)# ip address 192.168.20.1 255.255.255.0

MC_router(config)# interface GigabitEthernet0/1.30
MC_router(config-subif)# encapsulation dot1q 30
MC_router(config-subif)# ip address 192.168.30.1 255.255.255.0

MC_router(config)# interface GigabitEthernet0/1.100
MC_router(config-subif)# encapsulation dot1q 100
MC_router(config-subif)# ip address 192.168.100.1 255.255.255.0
MC_router(config-subif)# end
MC_router# write memory
```

<p align="center">
  <img src="https://i.imgur.com/TNmIA7W.png" width="750" alt="Configuring the VLAN 10 router subinterface">
</p>

<p align="center">
  <img src="https://i.imgur.com/O45WoMH.png" width="750" alt="Configuring the VLAN 20 router subinterface">
</p>

<p align="center">
  <img src="https://i.imgur.com/IQ4pjuq.png" width="750" alt="Configuring the VLAN 30 router subinterface">
</p>

<p align="center">
  <img src="https://i.imgur.com/SfVWJX8.png" width="750" alt="Configuring the VLAN 100 router subinterface">
</p>

### 4. Verified the Trunk and Routed Interfaces

The router showed all four subinterfaces in an up/up state.

<p align="center">
  <img src="https://i.imgur.com/qHSWSq9.png" width="750" alt="Verifying router subinterfaces with show ip interface brief">
</p>

The switch reported FastEthernet0/1 as an active 802.1Q trunk carrying VLANs 10, 20, 30, and 100.

<p align="center">
  <img src="https://i.imgur.com/VZrUjRt.png" width="750" alt="Verifying the 802.1Q trunk with show interfaces trunk">
</p>

### 5. Tested Inter-VLAN Connectivity

I added the appropriate default gateway to each endpoint.

<p align="center">
  <img src="https://i.imgur.com/EXBsaij.png" width="750" alt="Assigning a VLAN default gateway to a campus endpoint">
</p>

Pings between the user VLANs and the Server VLAN then succeeded. The first packet loss visible in one test is consistent with initial address-resolution activity; the repeated tests completed successfully.

<p align="center">
  <img src="https://i.imgur.com/VtU3iCk.png" width="750" alt="Successful inter-VLAN pings after router-on-a-stick configuration">
</p>

<p align="center">
  <img src="https://i.imgur.com/F17Jd6u.png" width="750" alt="Successful ping between a server and an endpoint in another VLAN">
</p>

## Phase 3: Deployed DHCP for the Campus VLANs

### 1. Established the Pre-Configuration Baseline

Before the DHCP pools were fully configured, a client request failed and the host assigned itself an APIPA address. This screenshot provides a useful before-state for the later DHCP validation.

<p align="center">
  <img src="https://i.imgur.com/6nkQMoD.png" width="750" alt="Client showing DHCP failure and an APIPA address before DHCP configuration">
</p>

### 2. Created DHCP Pools

I configured DHCP pools for VLANs 10, 20, and 30.

```text
MC_router(config)# ip dhcp pool VLAN10
MC_router(dhcp-config)# network 192.168.10.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.10.1
MC_router(dhcp-config)# exit

MC_router(config)# ip dhcp pool VLAN20
MC_router(dhcp-config)# network 192.168.20.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.20.1
MC_router(dhcp-config)# exit

MC_router(config)# ip dhcp pool VLAN30
MC_router(dhcp-config)# network 192.168.30.0 255.255.255.0
MC_router(dhcp-config)# default-router 192.168.30.1
MC_router(dhcp-config)# end
MC_router# write memory
```

<p align="center">
  <img src="../images/vlan-routing/21.png" width="750" alt="Creating DHCP pools for VLANs 10, 20, and 30">
</p>

### 3. Verified DHCP Bindings and Pool Utilization

`show ip dhcp binding` displayed dynamically leased client addresses.

<p align="center">
  <img src="../images/vlan-routing/22.png" width="750" alt="Verifying dynamically assigned addresses with show ip dhcp binding">
</p>

`show ip dhcp pool` displayed the configured scopes and lease statistics.

<p align="center">
  <img src="../images/vlan-routing/23.png" width="750" alt="Verifying DHCP pool utilization and address ranges">
</p>

### 4. Retested Routed Connectivity

DHCP-addressed endpoints successfully reached devices in other VLANs after inter-VLAN routing was in place.

<p align="center">
  <img src="../images/vlan-routing/24.png" width="750" alt="Successful cross-VLAN ping from a DHCP-addressed endpoint">
</p>

<p align="center">
  <img src="../images/vlan-routing/25.png" width="750" alt="Second successful cross-VLAN ping after DHCP configuration">
</p>

### 5. Added a DNS Option and Documented Address Exclusions

I added `8.8.8.8` as the DNS option for the campus DHCP pools. The client screenshot confirms that the DNS value was received, but the topology does not include simulated internet connectivity or a DNS query test.

```text
MC_router(config)# ip dhcp pool VLAN10
MC_router(dhcp-config)# dns-server 8.8.8.8
```

<p align="center">
  <img src="../images/vlan-routing/26.png" width="750" alt="DHCP client receiving the configured DNS server address">
</p>

The source document specifies that addresses `.1` through `.9` should be excluded from the VLAN 10, 20, and 30 pools.

```text
MC_router(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.9
MC_router(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.9
MC_router(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.9
```

<p align="center">
  <img src="../images/vlan-routing/27.png" width="750" alt="Campus endpoint addressing shown alongside the documented DHCP exclusion task">
</p>

## Phase 4: Added Wireless and Printer Endpoints

### 1. Added a Wireless Router

I added a Packet Tracer wireless router to VLAN 10 and configured its internet-facing connection to receive an address automatically.

<p align="center">
  <img src="../images/vlan-routing/28.png" width="750" alt="Selecting a wireless router for the Administration VLAN">
</p>

The device received `192.168.10.2` and its internal DHCP service was left enabled for the wireless clients.

<p align="center">
  <img src="../images/vlan-routing/29.png" width="750" alt="Wireless router receiving an address and providing a local DHCP scope">
</p>

I configured the SSID **AdminWifi** with WPA2-PSK authentication and AES encryption.

<p align="center">
  <img src="../images/vlan-routing/30.png" width="750" alt="Configuring AdminWifi with WPA2-PSK and AES">
</p>

### 2. Connected the Laptop and Tablet

<p align="center">
  <img src="../images/vlan-routing/31.png" width="750" alt="Laptop connected to AdminWifi">
</p>

<p align="center">
  <img src="../images/vlan-routing/32.png" width="750" alt="Tablet connected to AdminWifi">
</p>

### 3. Added Departmental Printers

<p align="center">
  <img src="../images/vlan-routing/33.png" width="750" alt="VLAN 10 printer configured with 192.168.10.5">
</p>

<p align="center">
  <img src="../images/vlan-routing/34.png" width="750" alt="VLAN 20 printer configuration">
</p>

<p align="center">
  <img src="../images/vlan-routing/35.png" width="750" alt="VLAN 30 printer configuration">
</p>

<p align="center">
  <img src="../images/vlan-routing/36.png" width="750" alt="Expanded campus topology">
</p>

## Module Validation Summary

| Test | Result | Evidence |
|---|---|---|
| VLAN creation and access-port assignment | Passed | VLAN and interface configuration captured |
| Same-VLAN connectivity | Passed | VLAN 10 hosts exchanged ICMP replies |
| Cross-VLAN isolation before routing | Passed | Inter-VLAN ping failed as expected |
| 802.1Q trunk | Passed | FastEthernet0/1 shown operating as a trunk |
| Router subinterfaces | Passed | Four gateway subinterfaces shown up/up |
| Inter-VLAN routing | Passed | User and server VLANs exchanged traffic |
| Campus DHCP | Passed | Pool statistics and client bindings displayed |
| DNS option delivery | Passed | Client received `8.8.8.8`; name resolution was not tested |
| Wireless association | Passed | Laptop and tablet joined `AdminWifi` |
| Printer addressing | Partially configured | VLAN 20 and VLAN 30 captures require correction |

## Technical Notes

- Cisco IOS rejected VLAN names containing spaces, so underscores were used.
- DHCP exclusion commands belong in global configuration mode.
- A production wireless design should avoid overlapping DHCP services.
- The VLAN 20 and VLAN 30 printers should use `.5` as the interface address and `.1` as the gateway.

## Module Outcome

This stage established the campus switching, routing, and addressing foundation. It demonstrated the difference between Layer 2 segmentation and Layer 3 communication, then extended the environment with automatically addressed wired and wireless endpoints.

## Continue the Project

[Next: Network Security with ACLs and Port Security →](../02-network-security/README.md)
