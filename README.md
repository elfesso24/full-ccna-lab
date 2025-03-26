# Enterprise Network Topology Documentation





written by : Mohammed Furrara , Feras Gsiea

## Scenario Overview
This network design represents a medium-sized enterprise with three primary functional areas:

1. **outer Network**
2. **inner network**
3. **Datacenter**


#### Key Requirements Addressed:
- Basic configuration 
- VLAN segmentation for different departments
- Redundant gateway services using HSRP
- OSPF routing between core devices
- NAT for internet connectivity
- Basic network security controls
- STP
- EtherChannel
- static Route
- DHCP
- SNMP
- ACL


## Topology :


![top.png](https://eraser.imgix.net/workspaces/kBBIhpN4Ob6RtMtTOT7F/3Tl3y7k0LGPg1Pb3jedwFSY7RAm1/7M43y8nDYV0Xwl2xm-ZGl.png?ixlib=js-3.7.0 "top.png")





![site 2.png](https://eraser.imgix.net/workspaces/kBBIhpN4Ob6RtMtTOT7F/3Tl3y7k0LGPg1Pb3jedwFSY7RAm1/DbII1g2kPZnFv2hLUyHzb.png?ixlib=js-3.7.0 "site 2.png")



---

## Network Design Overview
### 1. Layer 2/Layer 3 Infrastructure
| Device | Role | Key Features |
| ----- | ----- | ----- |
| MULTI1 | Core Switch | VLAN routing, Port-channel, STP root |
| MULTI2 | Core Switch | IP routing, ACLs, NAT |
| Switch | Access Switch | Basic Layer 2 connectivity |
| CENTER_ROUTER | Edge Router | OSPF, Dual WAN, NAT |
| R1/R2 | Distribution Routers | HSRP, OSPF, VLAN routing |
### 2. VLAN Configuration
| VLAN ID | Purpose | Gateway IP | HSRP Configuration |
| ----- | ----- | ----- | ----- |
| 10 | IT MANAGEMENT | 192.168.10.1 | Priority 150 (R1), Preempt |
| 20 | STAFF | 192.168.20.1 | - |
| 30 | HR | 192.168.30.1 | - |
### 3. IP Addressing Scheme
**Core Network:**

- 192.168.10.0/24 - IT_DEP
- 192.168.20.0/24 - Management 
- 192.168.30.0/24 - HR
**WAN Connections:**

- 11.0.0.0/29 - CENTER_ROUTER to R1
- 11.0.1.0/29 - CENTER_ROUTER to R2
---



## Spanning Tree Configuration :
**Purpose:**
 STP prevents Layer 2 loops that can cause broadcast storms and network instability by ensuring there is only one active path between any two network devices. It does this by electing a root bridge and selectively blocking redundant links until they are needed for failover.



#### 1. **STP Mode**
Both switches use the command:

```
spanning-tree mode pvst
```
This indicates that they are operating in Cisco’s PVST mode, where a separate spanning tree instance is maintained for each VLAN.



#### 2. **Priority Settings for VLANs 10, 20, and 30**
- **MULTI1 :**
 The configuration includes the command:
```
spanning-tree vlan all priority 24576 
```
This sets a lower priority (24576) for VLANs 10, 20, and 30. In STP, a lower priority increases the likelihood that the switch will be elected as the root bridge for these VLANs. Essentially, MULTI1 is being prioritized as the primary root for these VLANs if no other switch has an even lower priority. 



- **MULTI2 (Switch with hostname MULTI2):**
 The configuration on this switch shows:
```
spanning-tree vlan all priority 28672
```
Here, the priority is set to 28672, which is higher than MULTI1’s value. In a spanning tree election, a higher priority makes it less likely for the switch to become the root bridge. This configuration positions MULTI2 as a backup or secondary candidate for the root bridge role in these VLANs. 



#### 3. **Additional STP Safeguards**
Both switches include extra commands on specific interfaces to further secure the spanning tree topology:

- **BPDU Guard:**
 On interfaces such as Port-channel1, the command is used to disable a port if it unexpectedly receives Bridge Protocol Data Units (BPDUs). This protects the network from misconfigured devices or potential security 
```
spanning-tree bpduguard enable
```
---

## VLAN Configuration :




**On MULTI1 :**

- **SVI Creation:**
 The configuration creates interfaces for VLANs: 
    - **VLAN10:**
 Configured with IP address **192.168.10.1/24**.
    - **VLAN20:**
 Configured with IP address **192.168.20.1/24**.
    - **VLAN30:**
 Configured with IP address **192.168.30.1/24**.
- **Default VLAN:**
 The default interface for VLAN1 is present but shutdown and has no IP address.
- **Port Assignments:**
 Specific physical interfaces GigabitEthernet1/0/6 and 1/0/7 are configured in access or trunk modes to carry traffic for these VLANs.


**On MULTI2 :**

- **SVI Creation:**
 Similar to MULTI1, the switch defines: 
    - **VLAN10:**
 Configured with IP address **192.168.10.1/24**.
    - **VLAN20:**
 Configured with IP address **192.168.20.1/24**.
    - **VLAN30:**
 Configured with IP address **192.168.30.1/24**.
- **Default VLAN:**
 Again, VLAN1 is defined but shutdown.


---

## EtherChannel :
EtherChannel is a technology that bundles multiple physical links into a single logical link. This aggregation increases available bandwidth and provides redundancy, so if one link fails, the others continue to carry traffic.



**MULTI1 :**

- **Logical Interface – Port-channel1:**
 The configuration defines a logical interface:
```
interface Port-channel1
 switchport mode trunk
 spanning-tree bpduguard enable
 spanning-tree guard root
```
This Port-channel1 is set as a trunk, meaning it carries multiple VLANs across the aggregated link. The spanning tree safeguards (BPDU Guard and Root Guard) are applied to ensure stability and prevent unauthorized topology changes.



- **Physical Interfaces Assigned to EtherChannel:**
 Two physical interfaces are added to this EtherChannel using LACP (Link Aggregation Control Protocol) in active mode:

```
interface range GigabitEthernet1/0/4-5
switchport mode trunk
channel-group 1 mode active
```
#### Note : the same configuration to the same interfaces was implemented on MULTI2
---

## DHCP:
DHCP Server was set in a server based on the requirements of the assignment.

it has 3 pools :

1.server pool:

![Screenshot 2025-03-10 172205.png](https://eraser.imgix.net/workspaces/kBBIhpN4Ob6RtMtTOT7F/3Tl3y7k0LGPg1Pb3jedwFSY7RAm1/GiyYermJdpW9Uoj90u14P.png?ixlib=js-3.7.0 "Screenshot 2025-03-10 172205.png")



2.INNER_DATA_DEVICES

![Screenshot 2025-03-10 172354.png](https://eraser.imgix.net/workspaces/kBBIhpN4Ob6RtMtTOT7F/3Tl3y7k0LGPg1Pb3jedwFSY7RAm1/lzq8gp4t9EeT8lOXSikoO.png?ixlib=js-3.7.0 "Screenshot 2025-03-10 172354.png")



3.HR

![Screenshot 2025-03-10 172305.png](https://eraser.imgix.net/workspaces/kBBIhpN4Ob6RtMtTOT7F/3Tl3y7k0LGPg1Pb3jedwFSY7RAm1/kpoM0pItqL1E92jrlFir3.png?ixlib=js-3.7.0 "Screenshot 2025-03-10 172305.png")



---

## SNMP :
**Simple Network Management Protocol** (SNMP) is an Internet Standard protocol for collecting and organizing information about managed devices on IP networks and for modifying that information to change device behaviour.



#### SNMP was Set in the two main compass networks (main and inner network)
#### Configuration :
```
snmp-server community read ro
snmp-server community write rw
```
the snmp server is then accessed by the pc using the mib browser



---

## Security :
1.ssh enabled on the routers and the multi layer switches

- configure hostname 
```
enable
conf t
hostname { hostname to set to the device }
username feras privilege 15 secret 123
```
- enable ssh
```
line vty 0 4
transport input ssh
login local
exit
```
2.passwords on the enable and the global configuration mode 

- in enable mode
```
enable secret 123
```
- in configuration mode 
```
 line console 0
 password console_password
 login
 exit
```
#### NOTE :Channel port was enabled to prevent **MAC Flooding Attacks ** 
---

## OSPF : 


OSPF (Open Shortest Path First) is a link-state routing protocol that dynamically exchanges routes within an autonomous system. Key configurations include:

- **Network advertisement**: Using `network`  commands with wildcard masks.
- **Default route propagation**: Enabled via `default-information originate` .


### **Router Configurations**
1. **CENTER_ROUTER (From the datacenter)**
    - **OSPF Process**: ID 1.
    - **Advertised Networks**:
        - `11.0.0.0/29`  (Serial0/0/0) and 
        - `11.0.1.0/29`  (Serial0/0/1) in Area 0.
    - **Default Route**: `default-information originate`  with no static default route configured.
2**. R1 **

- **OSPF Process**: ID 1.
- **Advertised Networks**:
        `11.0.0.0/29`  (Serial0/0/0).

- **Default Route**: `default-information originate`  with no static route.
3. **R2 (Router5)**

- **OSPF Process**: ID 1.
- **Advertised Networks**:
        `11.0.1.0/29`  (Serial0/0/1).

---

## HSRP :


HSRP (Hot Standby Router Protocol) is a Cisco redundancy protocol that allows multiple routers to act as a single virtual router, ensuring seamless failover for hosts. Key elements include:

- **Virtual IP**: Shared IP used by devices as their default gateway.
- **Priority**: Determines the active router (higher priority wins).
- **Preemption**: Allows a higher-priority router to reclaim the active role after recovery.


**HSRP Configuration in the Routers**

1. **R1 **
    - **Interface**: `GigabitEthernet0/0.10`  (VLAN 10).
    - **Virtual IP**: `192.168.10.2`  
    - **HSRP Group 1**:
        - `standby 1 ip 192.168.10.2` 
        - `standby 1 priority 150` 
        - `standby 1 preempt` 
2. **R2 **
    - **Interface**: `GigabitEthernet0/0.10`  (VLAN 10).
    - **Virtual IP**: `192.168.10.1`  (same as R1’s physical IP).
    - **HSRP Group 1**:
        - `standby 1 ip 192.168.10.1`  ➜ Matches R1’s virtual IP.
        - `standby 1 preempt`  ➜ Preemption enabled (default priority: **100**).


---

## **NAT : **
**Purpose**: Translates private internal IPs to public IPs for external communication.

**Configuration Details**:

1. **CENTER_ROUTER (Router0)**:
    - **Command**:
```
 ip nat inside source list 2 interface Serial0/0/1 overload   
```
- **Translation**: Internal IPs (defined by ACL 2) are translated to the IP of `Serial0/0/1`  using PAT.
- **Interfaces**:
    - **Inside**: `GigabitEthernet0/0`  (192.168.10.1).
    - **Outside**: `Serial0/0/1`  (11.0.1.1).


2.**R1 (Router1)**:

- **Command**:
```
 ip nat inside source list 2 interface Serial0/0/0 overload 
```
- **Translation**: Traffic from ACL 2 is translated to the IP of `Serial0/0/0`  (11.0.0.2).
- **Interfaces**:
    - **Inside**: `GigabitEthernet0/0.10`  (192.168.10.1).
    - **Outside**: `Serial0/0/0` .


3.**R2 (Router5)**:

- **Command**: 
```
ip nat inside source list 2 interface Serial0/0/1 overload 
```
- **Translation**: Internal traffic is mapped to `Serial0/0/1`  (11.0.1.2).
- **Interfaces**:
    - **Inside**: `GigabitEthernet0/0.10`  (192.168.10.1).
    - **Outside**: `Serial0/0/1` .


---

## ACL :
**Purpose**: Defines which internal IPs are permitted to use NAT.

**Configuration Details**:

- **ACL 2** (Identical on all routers):
    - **Command**:
```
 access-list 2 permit 192.168.10.0 0.0.0.255 
```
- **Function**: Permits traffic from the **192.168.10.0/24** subnet for NAT translation.


