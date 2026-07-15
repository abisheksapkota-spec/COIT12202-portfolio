<img width="808" height="774" alt="GNS3-Intro-12312491-network" src="https://github.com/user-attachments/assets/096649f4-527b-42e4-9249-f6363bc13ee8" />
<img width="1390" height="808" alt="GNS3-Intro-12312491-ipaddress" src="https://github.com/user-attachments/assets/74ed254c-dd7e-4d7b-8510-b31e5c8fc693" />

1.	The Usage of /etc/network/interfaces

The /etc/network/interfaces file serves the purpose of configuring all the network interfaces in the Debian/Ubuntu operating system, which includes the configuration of IP addresses, subnet masks, and gateways, and whether the interface is connected via DHCP or static IP address. This file is read and its configurations are applied automatically once the system boots up.
There are reasons why it is recommended to configure this file before starting the node:
-	This configuration is permanent and remains even after reboot.
-	Interfaces are automatically configured upon boot.
-	There is no need for manual configuration upon every restart of GNS3 node.

2.	What is a loopback interface (lo) 

The loopback interface allows a machine to communicate with itself. Its default IPv4 address is 127.0.0.1. This loopback interface is used for:
-	Testing network stack locally.
-	Communicating with the applications that are on the same machine.

Loopback interfaces cannot be assigned static LAN addresses since they do not take part in the network. It is only interfaces like eth0 that connect to the network and therefore a static IP address (10.10.1.101/24) must be assigned to eth0.

3.	Benefits of using virtual nodes (GNS3) instead of hardware
Some advantages of GNS3 include:
-	Reduced expense: There is no requirement to purchase any physical networking hardware.
-	Simple testing: The ability to create, change, or reboot networks easily.
-	Secure environment: Perform experiments without interfering with any live networks.
-	Labs that can be replicated: Saving and loading the same topology as many times as required.

The main shortcoming of GNS3 is the lack of practical experience with physical devices and wiring.
