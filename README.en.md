# Network Security with EVE-NG

<img width="940" height="467" alt="Topology" src="https://github.com/user-attachments/assets/68fe9723-bc08-42af-8f21-d21f5ca72667" />

### Description

Implementation of a network topology segmented into 3 VLANs, with inter-VLAN
routing on a Cisco router using access control lists (ACLs) to apply security
policies, blocking unauthorized traffic between segments and controlling
internet access. The objective of this project was to **practice the principles
of network segmentation and least-privilege access control**.

---

### Topology

Implementation of three VLANs with:

- 3 PCs (one in each VLAN)  
- 2 Switches  
- 1 Router  
- 1 Network Bridge (Cloud)  

---

### Requirements

- VMware Workstation Pro  
- EVE-NG (4 GB RAM, 2 CPUs, 20 GB HD, NAT mode)  
- Cisco IOL 3725  
- WinSCP  
- EVE-NG Integration Package  

---

### Network Structure

| VLAN | Subnet            | Example Host       | Description             |
|------|-------------------|-------------------|-------------------------|
| 10   | 192.168.10.0/24   | 192.168.10.10     | Internal Network 1      |
| 20   | 192.168.20.0/24   | 192.168.20.10     | Internal Network 2      |
| 30   | 192.168.30.0/24   | 192.168.30.10     | Administrative Network  |

---

### ACLs (Access Control)

```bash
ip access-list extended VLAN-RESTRICOES
 ! VLAN 10 cannot access VLAN 20
 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255

 ! VLAN 20 cannot access VLAN 10 or the internet
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.20.0 0.0.0.255 any

 ! VLAN 30 has full access
 permit ip 192.168.30.0 0.0.0.255 any

 ! All other traffic is denied by default
 deny ip any any
```

---

## Router Configuration with ACLs and NAT

```plaintext
enable
conf t

int fa1/0
ip address dhcp
no shut
exit

int fa0/0
no shut
exit

int fa0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
ip nat inside
exit

int fa2/0
no shut
exit

int fa2/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
ip nat inside
exit

int fa2/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
ip nat inside
exit

ip access-list standard NAT-VLANS
permit 192.168.10.0 0.0.0.255
permit 192.168.20.0 0.0.0.255
permit 192.168.30.0 0.0.0.255
exit

ip nat inside source list NAT-VLANS interface fa1/0 overload

ip access-list extended VLAN-RESTRICOES
deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 any
permit ip 192.168.30.0 0.0.0.255 any
deny ip any any
exit

int fa0/0.10
ip access-group VLAN-RESTRICOES in
exit
int fa2/0.20
ip access-group VLAN-RESTRICOES in
exit

ip domain-lookup
ip name-server 8.8.8.8
ip name-server 1.1.1.1

end
wr
```

---

## Switch 1 Configuration with Access VLANs

```plaintext
enable
conf t

vlan 10
name VLAN10
exit

int fa1/0
switchport mode access
switchport access vlan 10
no shut
exit

int fa1/15
switchport mode trunk
switchport trunk allowed vlan all
no shut
exit

end
wr
```

---

## Switch 2 Configuration with Trunk and VLANs

```plaintext
enable
conf t

vlan 20
name VLAN20
exit

vlan 30
name VLAN30
exit

int fa1/0
switchport mode access
switchport access vlan 20
no shut
exit

int fa1/1
switchport mode access
switchport access vlan 30
no shut
exit

int fa1/15
switchport mode trunk
switchport trunk allowed vlan all
no shut
exit

end
wr
```

---

## PC1 Configuration (VLAN 10)

```plaintext
enable
conf t
int fa0/0
ip address 192.168.10.10 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 192.168.10.1
ip name-server 8.8.8.8
end
wr
```

---

## PC2 Configuration (VLAN 20)

```plaintext
enable
conf t
int fa0/0
ip address 192.168.20.10 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 192.168.20.1
ip name-server 8.8.8.8
end
wr
```

---

## PC3 Configuration (VLAN 30)

```plaintext
enable
conf t
int fa0/0
ip address 192.168.30.10 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 192.168.30.1
ip name-server 8.8.8.8
end
wr
```

---

## Result

This project demonstrated in practice how to use VLANs, inter-VLAN routing, and ACLs to segment a network and apply basic security rules. Each VLAN represents a separate segment, allowing control over which networks can communicate with each other.

Using ACLs, it was possible to block unauthorized traffic between VLANs and allow access only when necessary. NAT allowed the internal networks to use the external connection while keeping private addresses inside the network.

The main goal of this project was to practice fundamental networking and security concepts such as **segmentation, access control, and traffic organization** in simulated environments using EVE-NG.
