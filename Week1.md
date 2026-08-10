

# GNS3 Introduction: Network Configuration Basics

This document covers three key concepts related to setting up and understanding a GNS3 network lab: the `/etc/network/interfaces` file, the loopback interface, and the benefits of virtual nodes over physical hardware.


## 1. The Usage of `/etc/network/interfaces`

The `/etc/network/interfaces` file serves the purpose of configuring **all network interfaces** in the Debian/Ubuntu operating system. This includes:

- IP addresses
- Subnet masks
- Gateways
- Whether the interface is configured via **DHCP** or a **static IP address**

This file is read and its configuration is **applied automatically** whenever the system boots up.

> [!TIP]
> It is recommended to configure this file *before* starting the node, for the following reasons:
> - The configuration is **permanent** and remains in place even after a reboot.
> - Interfaces are **automatically configured** on boot.
> - There is **no need for manual configuration** every time the GNS3 node restarts.



## 2. What is a Loopback Interface (`lo`)?

The **loopback interface** allows a machine to communicate with itself. Its default IPv4 address is:

```
127.0.0.1
```

This loopback interface is used for:

- **Testing** the network stack locally
- **Communicating** with applications running on the same machine

> [!NOTE]
> Loopback interfaces **cannot** be assigned static LAN addresses, since they do not take part in the network.
> Only interfaces like `eth0` connect to the actual network, and it is `eth0` that must be assigned a static IP address, e.g. `10.10.1.101/24`.



## 3. Benefits of Using Virtual Nodes (GNS3) Instead of Hardware

Some advantages of GNS3 include:

- **Reduced expense**: no need to purchase physical networking hardware
- **Simple testing**: easily create, change, or reboot networks
- **Secure environment**: experiment without affecting any live network
- **Replicable labs**: save and load the same topology as many times as needed

> [!WARNING]
> The main shortcoming of GNS3 is the **lack of practical, hands-on experience** with physical devices and cabling.



### Reference Diagrams

<img width="808" height="774" alt="GNS3-Intro-12312491-network" src="https://github.com/user-attachments/assets/096649f4-527b-42e4-9249-f6363bc13ee8" />

*Figure 1: GNS3-Intro-12312491-network.png*

<img width="1390" height="808" alt="GNS3-Intro-12312491-ipaddress" src="https://github.com/user-attachments/assets/74ed254c-dd7e-4d7b-8510-b31e5c8fc693" />

*Figure 2: GNS3-Intro-12312491-ipaddress.png*

### Activities:

<img width="1366" height="886" alt="image" src="https://github.com/user-attachments/assets/e747add4-8dfd-4354-86fa-afa1b53cebf8" />

<img width="1384" height="754" alt="image" src="https://github.com/user-attachments/assets/a2471f25-f221-48c6-8f1f-ab4a1263e967" />

<img width="1366" height="760" alt="image" src="https://github.com/user-attachments/assets/a5f19139-23da-44a5-89ed-bde5c0fbdabb" />

<img width="1374" height="776" alt="image" src="https://github.com/user-attachments/assets/6f2eb469-a185-4e01-82d9-65d38b4a8ea5" />




