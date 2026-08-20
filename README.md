# networkwalks-B082-week2-Cybersecurity-Footprinting-Scanning

A practical cybersecurity lab covering footprinting using Maltego and network scanning using Zenmap. The activities focused on understanding reconnaissance, discovering publicly available information, identifying live hosts on a controlled virtual network, and documenting the results.

---

## Week 2 Objectives

The main objectives of this week's activities were:

- Perform domain footprinting using Maltego.
- Understand how publicly available information can be discovered through OSINT.
- Use Maltego transforms to identify information associated with a domain.
- Configure and use Zenmap for network scanning.
- Identify the local subnet of the cybersecurity lab.
- Discover live hosts on the controlled virtual network.
- Identify IP addresses and MAC addresses of discovered hosts.
- Generate a network topology using Zenmap.
- Document the activities, findings, and observations.

---

## Modules Completed

### Elective Module

#### W2-PM3 — Maltego-based Footprinting

Maltego was used to perform domain-based footprinting against the assigned target:

```
networkwalks.com
```

The objective was to understand how publicly available information associated with a domain can be discovered and represented using Maltego.

**Tools Used**
- Maltego Graph Desktop 4.12.1
- Maltego Transforms — Search Engine transform

**Procedure**
1. Installed and configured Maltego on Windows.
2. Created a new Maltego graph.
3. Added a Domain entity to the graph.
4. Configured the Domain entity as `networkwalks.com`.
5. Selected an email-related transform.
6. Executed `To Email Addresses [Search Engine]`.
7. Analyzed the resulting entity and relationship.

**Result**

The transform returned one email entity associated with the domain:

```
info@networkwalks.com
```

The resulting Maltego graph:

```
networkwalks.com
       |
       v
info@networkwalks.com
```

The final graph contained:
- 2 entities
- 1 link

The transform output also showed that one entity was returned from the domain.

**Security Relevance**

Publicly discoverable organizational information can provide useful information during reconnaissance. An exposed organizational email address can contribute to an understanding of the publicly visible footprint of an organization. Security teams can use this type of information to identify unnecessary information exposure and review what information is available to external users.

The discovery of an email address alone does not indicate the presence of a vulnerability. It is an information-discovery result that may be considered during a broader security assessment.

**Evidence Captured**
- Maltego main interface
- Domain entity configured with `networkwalks.com`
- Email-related transform execution
- `info@networkwalks.com` result
- Transform output showing one entity returned
- Final graph showing the relationship between the domain and email entity
  
### Search Web Footprinting

I also used the **Search Web [Search Engine]** transform against the `networkwalks.com` domain.

The transform returned **21 entities**, producing a graph containing **22 entities and 21 links** in total.

The results represented publicly indexed web resources associated with the search query. These results were treated as search-engine findings and were not assumed to be assets owned by the target without further verification.

This activity helped demonstrate how search-engine-based OSINT can be used to understand an organization's publicly visible information footprint.

**Evidence Captured**
- Search Web transform execution
- Search Web results showing 21 returned entities
- Final Maltego graph showing 22 entities and 21 links

---

### Essential Module

#### W2-PM5 — Zenmap-based Network Scanning

Zenmap was used to perform network discovery on the controlled VirtualBox Host-Only network. The purpose of this activity was to identify active hosts on the lab network and collect basic information about the discovered systems.

**Network Configuration**

The Windows host initially had the following Host-Only network configuration:

| Setting | Value |
|---|---|
| IPv4 Address | 192.xxx.xx.x |
| Subnet Mask | 255.255.255.0 |

Therefore, the Host-Only network used for the scanning exercise was:

```
192.xxx.xx.x/24
```

The original `10.0.0.0/24` lab network was preserved.

**Virtual Lab Network**

The lab was configured using multiple virtual network adapters.

*Original Lab Network*

```
Kali Linux (10.0.0.2/24)
        |
   NAT Network (10.0.0.0/24)
        |
Windows 10 VM (10.0.0.10/24)
```

*Host-Only Network*

A second Host-Only interface was added so that the Windows host and virtual machines could participate in the same controlled scanning network.

```
                    Windows Host
                    192.xxx.xx.x
                         |
                  Host-Only Network
                  192.xxx.xx.x/24
                    /           \
                   /             \
             Kali Linux       Windows 10 VM
            192.xxx.xx.xx    192.xxx.xx.xxx
```

Kali therefore had two network interfaces:

```
eth0 -> 10.0.0.2/24
eth1 -> 192.xxx.xx.x/24
```

The Windows 10 VM had:

```
Ethernet   -> 10.0.0.10/24
Ethernet 2 -> 192.xxx.xx.x/24
```

This allowed the existing lab network to remain intact while using the Host-Only network for controlled network discovery.

**Zenmap Ping Scan**

| Parameter | Value |
|---|---|
| Target network | 192.xxx.xx.x/24 |
| Zenmap profile | Ping Scan |
| Nmap command | `nmap -sn 192.xxx.xx.x/24` |

The `-sn` option was used for host discovery without performing a port scan.

**Live Host Discovery**

The Zenmap scan identified active hosts on the Host-Only network. The scan output reported:

```
3 hosts up
```

The discovered lab addresses included:
- 192.xxx.xx.x
- 192.xxx.xx.xx
- 192.xxx.xx.xxx

The VirtualBox DHCP infrastructure was also visible at `192.xxx.xx.x`.

The Zenmap results were reviewed to distinguish the virtual network infrastructure from the lab machines.

**Discovered Systems**

| System | IP Address | Role |
|---|---|---|
| Windows Host | 192.xxx.xx.x | Physical host |
| VirtualBox DHCP | 192.xxx.xx.x | Virtual network infrastructure |
| Kali Linux | 192.xxx.xx.x | Security testing VM |
| Windows 10 VM | 192.xxx.xx.xxx | Lab target |

MAC address information was also displayed by Zenmap for the discovered hosts and was captured in the scan evidence.

**Network Topology**

Zenmap's Topology feature was used after the host discovery scan. The topology provided a graphical representation of the discovered systems on the network and was saved as a PDF for inclusion in the final project documentation:

```
W2_PM5_Zenmap_Topology.pdf
```

---

## Problems Faced and Solutions

### 1. Kali and Windows were initially on different subnets

**Problem**

Initially, Kali was configured with `10.0.0.2/24`, while the Windows host's Host-Only adapter was `192.xxx.xx.x/24`. Therefore, when Zenmap initially scanned `192.xxx.xx.x/24`, only the Windows host was discovered.

**Solution**

Instead of changing the existing `10.0.0.0/24` lab network, a second network adapter was added to Kali. The second adapter was configured as a Host-Only Adapter. Kali then used:

```
eth0 -> 10.0.0.2/24
eth1 -> 192.xxx.xx.x/24
```

The original `10.0.0.0/24` network was preserved.

### 2. Kali's Host-Only interface did not initially receive an IPv4 address

**Problem**

After adding the Host-Only adapter, Kali detected the new interface as `eth1`. However, it initially had no IPv4 address. NetworkManager showed the interface attempting to obtain IP configuration.

**Solution**

The VirtualBox Host-Only network and DHCP configuration were checked. The Host-Only network was configured as `192.xxx.xx.x/24` with DHCP enabled. Since Kali was unable to obtain an address automatically, a static address was configured for the Host-Only interface: `192.xxx.xx.x/24`. The connection was then successfully activated using NetworkManager.

**Result**

Kali successfully obtained `192.xxx.xx.x/24` and became visible to Zenmap during the subsequent network scan.

### 3. Windows VM needed to be connected to the Host-Only network

**Problem**

The Windows 10 VM initially had only its existing `10.0.0.10/24` network connection. Therefore, it was not part of the `192.168.56.0/24` network being scanned by Zenmap.

**Solution**

A second network adapter was added to the Windows 10 VM and connected to the same Host-Only network. The Windows VM then received `192.168.56.100/24`, allowing it to appear as a live host during the Zenmap scan.

**Verification**

The final network configuration was verified using `ip addr` on Kali and `ipconfig` on Windows.

| Machine | Interface | Address |
|---|---|---|
| Kali | eth0 | 10.0.0.2/24 |
| Kali | eth1 | 192.xxx.xx.x/24 |
| Windows Host | Host-Only Adapter | 192.xxx.xx.x/24 |
| Windows 10 VM | Ethernet | 10.0.0.10/24 |
| Windows 10 VM | Ethernet 2 | 192.xxx.xx.xxx/24 |

**Zenmap**

| Parameter | Value |
|---|---|
| Target | 192.xxx.xx.x/24 |
| Profile | Ping Scan |
| Command | `nmap -sn 192.xxx.xx.x/24` |

The final scan successfully identified the active hosts on the Host-Only network.

---

## What I Learned This Week

**Footprinting and OSINT**
- Learned the purpose of footprinting during the reconnaissance phase.
- Learned how Maltego can be used for domain-based footprinting.
- Learned how a Domain entity can be used as a starting point for an investigation.
- Learned how Maltego transforms can retrieve related information.
- Learned how email addresses can be associated with a domain through search-based transforms.
- Learned how relationships between entities can be represented visually in Maltego.
- Understood that publicly available information can contribute to an organization's visible attack surface.
- Learned how search-engine-based transforms can reveal publicly indexed information related to a domain.
- Learned that OSINT results should be validated before assuming that a discovered resource is owned or controlled by the target.

**Network Scanning**
- Learned how Zenmap provides a graphical interface for Nmap.
- Learned how to identify the local subnet using `ipconfig`.
- Learned how CIDR notation such as `/24` represents a network range.
- Learned how a Ping Scan can be used to identify live hosts.
- Learned how IP and MAC address information can be viewed for discovered hosts.
- Learned how Zenmap can generate a graphical network topology.

**Virtual Networking**
- Learned the difference between a NAT Network and a Host-Only network.
- Learned how multiple network adapters can be assigned to a virtual machine.
- Learned how a VM can participate in multiple virtual networks.
- Learned how VirtualBox DHCP works on a Host-Only network.
- Learned how static IP configuration can be used when DHCP address assignment fails.
- Learned how network configuration affects the results of network scanning.

---

## Security Relevance

Footprinting and network scanning are important parts of the reconnaissance stage of a cybersecurity assessment. Footprinting helps identify information that is publicly available about a target, while network scanning helps identify active systems and network infrastructure.

```
Maltego -> Domain Footprinting -> Publicly Available Information
Zenmap  -> Live Host Discovery  -> Network Topology
```

These activities provide an initial understanding of a target's information and network exposure. However, the discovery of a host, IP address, email address, or other information does not automatically indicate a vulnerability. Further authorized testing would be required to validate any security weakness.

---

## Ethical and Safety Considerations

All network scanning performed during this project was limited to the controlled VirtualBox lab environment. The Zenmap scan targeted `192.xxx.xx.x/24`, which was configured as a Host-Only network for the cybersecurity lab. The lab contained controlled virtual machines used for educational testing.

Footprinting activities were performed within the scope of the assigned educational exercise and with appropriate permission.

> Unauthorized scanning, enumeration, or exploitation of systems that do not belong to the tester can have legal and security consequences.

---

## Tools Used

**Footprinting**
- Maltego Graph Desktop 4.12.1
- Maltego Transforms

**Network Scanning**
- Zenmap
- Nmap

**Operating Systems**
- Kali Linux
- Windows 10

**Virtualization**
- Oracle VirtualBox

---

## Evidence Collected

**W2-PM3 — Maltego**
- Maltego installation/setup
- Maltego main interface
- Domain entity — `networkwalks.com` configuration
- Email-related transform
- `info@networkwalks.com` result
- Transform output
- Final Maltego graph

**W2-PM5 — Zenmap**
- Windows `ipconfig` output
- Kali `ip addr` output
- VirtualBox Host-Only network configuration
- Zenmap Ping Scan
- Live-host discovery
- IP address information
- MAC address information
- Zenmap topology / network topology PDF

---

## Conclusion

During Week 2 of the Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting and network scanning.

For **W2-PM3**, Maltego was used to perform domain-based footprinting against `networkwalks.com`. A Domain entity was created, and both an email-related transform and the **Search Web [Search Engine]** transform were used. The email transform returned `info@networkwalks.com`, while the Search Web transform returned 21 entities and produced a graph containing 22 entities and 21 links.

For **W2-PM5**, Zenmap was used to perform a Ping Scan against the controlled `192.xxx.xx.x/24` Host-Only network. The lab network was configured with multiple virtual machines and network interfaces, allowing live hosts to be discovered and analyzed.

The network configuration and troubleshooting process was also an important part of the exercise. It helped demonstrate how subnet configuration, virtual network adapters, DHCP, static addressing, and host discovery are connected.

Overall, Week 2 provided practical experience with two important reconnaissance activities:

```
Footprinting / OSINT + Network Scanning -> Initial Attack Surface Understanding
```

---
