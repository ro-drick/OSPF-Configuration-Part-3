## README: OSPF Troubleshooting Packet Tracer Lab

### 1. **Network Diagram Overview**
This lab involves troubleshooting and extending an OSPF network. Key tasks include configuring a newly added serial connection between routers R1 and R2, fixing routing issues to a specific subnet, resolving OSPF neighbor adjacency problems, and troubleshooting PC connectivity to an external server (8.8.8.8).

<img src="https://github.com/ro-drick/OSPF-Configuration-Part-3/blob/main/ospf-3.PNG">

#### Objectives
The network has been pre-configured (IP addresses, OSPF)

1. The connection between R1 and R2 has been newly added.
  -Configure the serial connection between R1 and R2 (clock rate of 128000).
  -Configure OSPF on R1 and R2.

2. Only R3 has a route to 10.0.2.0/24.  Why?  Fix the problem.

3. R2 and R4 won't become OSPF neighbors with R5.  Why?  Fix the problem.

4. PC1 and PC2 can't ping the external server 8.8.8.8.  Why?  Fix the problem.

5. Examine the LSDB.  What LSAs are present?

### 2. **Addressing Scheme**
| **Device** | **Interface**  | **IP Address**       | **Subnet Mask**      | **Description**         |
|------------|----------------|----------------------|----------------------|-------------------------|
| R1         | G0/0           | 10.0.1.1             | 255.255.255.0         | Link to SW1             |
| R1         | S0/0/0         | 192.168.12.1         | 255.255.255.252       | Link to R2              |
| R2         | S0/0/0         | 192.168.12.2         | 255.255.255.252       | Link to R1              |
| R2         | G0/0           | 192.168.245.1        | 255.255.255.248       | Link to SW3             |
| R3         | G0/1           | 192.168.34.1         | 255.255.255.252       | Link to R4              |
| R4         | G0/1           | 192.168.34.2         | 255.255.255.252       | Link to R3              |
| PC1        | NIC            | 10.0.1.100           | 255.255.255.0         | Host connected to SW1   |
| PC2        | NIC            | 10.0.2.100           | 255.255.255.0         | Host connected to SW2   |

### 3. **Step-by-Step Configuration**

#### a. **Configure the Serial Connection between R1 and R2**
A new serial link between R1 and R2 needs to be configured with a clock rate of 128000 and added to OSPF.

**R1:**
```bash
R1(config)# interface serial0/0/0
R1(config-if)# ip address 192.168.12.1 255.255.255.252
R1(config-if)# clock rate 128000
R1(config-if)# no shutdown
R1(config)# router ospf 1
R1(config-router)# network 192.168.12.0 0.0.0.3 area 0
```

**R2:**
```bash
R2(config)# interface serial0/0/0
R2(config-if)# ip address 192.168.12.2 255.255.255.252
R2(config-if)# no shutdown
R2(config)# router ospf 1
R2(config-router)# network 192.168.12.0 0.0.0.3 area 0
```

#### b. **Fix the Routing Issue for 10.0.2.0/24**
Currently, only R3 has a route to 10.0.2.0/24. This may be due to OSPF network type misconfigurations on R3, particularly the incorrect use of the point-to-point network type. Removing this configuration and reverting it to broadcast will fix the issue.

**R3:**
```bash
R3(config)# interface gig0/1
R3(config-if)# ip ospf network broadcast
```

#### c. **Fix the OSPF Adjacency Issue between R2, R4, and R5**
R2 and R4 are not forming OSPF adjacencies with R5. The problem is caused by mismatched Hello and Dead intervals between the routers.

To fix this, ensure that the Hello and Dead intervals are consistent on R2, R4, and R5.

**R2 and R4:**
```bash
R2(config)# interface gig0/0
R2(config-if)# ip ospf hello-interval 10
R2(config-if)# ip ospf dead-interval 40

R4(config)# interface gig0/0
R4(config-if)# ip ospf hello-interval 10
R4(config-if)# ip ospf dead-interval 40
```

Ensure the same Hello and Dead intervals are configured on **R5**.

#### d. **Troubleshoot External Connectivity (PC1/PC2 to 8.8.8.8)**
PC1 and PC2 cannot ping the external server 8.8.8.8 because no default route has been configured for external connectivity. R2 must be configured to propagate a default route to the rest of the OSPF network.

**R2 (ASBR):**
```bash
R2(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
R2(config)# router ospf 1
R2(config-router)# default-information originate
```

#### e. **Examine the LSDB**
Check the Link-State Database (LSDB) on each router to verify that the correct LSAs are present.

```bash
R1# show ip ospf database
```

Typical LSAs you should see include:
- **Type 1** (Router LSAs): Generated by all routers.
- **Type 2** (Network LSAs): Generated by the Designated Router (DR).
- **Type 3** (Summary LSAs): Generated by Area Border Routers (ABRs).

### 4. **Conclusion**
This lab walks through troubleshooting key OSPF issues, including misconfigurations on serial links, network type mismatches, Hello/Dead interval mismatches, and missing default routes. Proper OSPF configuration and verification using LSDB will ensure that all routers and devices in the network have full connectivity.

## Acknowledgements


Special thanks to **Jeremy's IT Lab** for providing valuable resources and tutorials that greatly contributed to the completion of this exercise. His in-depth explanations and practical demonstrations have been instrumental in enhancing my understanding of Cisco networking concepts and the effective use of Packet Tracer.

For more information and additional resources, visit [Jeremy's IT Lab](https://jeremysitlab.com/) and check out his YouTube for the full course, [Jeremy's IT Lab Free CCNA 200-301 | Complete Course](https://www.youtube.com/playlist?list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ)
