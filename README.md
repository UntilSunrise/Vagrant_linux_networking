# Vagrant Linux Networking
## Task conditions
### Goal
The goal of this project is to set up a network configuration with three Ubuntu virtual machines (VMs) using Vagrant. Each VM will have specific network interfaces and configurations.

### VM A
- IP Addresses:
  - 172.16.0.0/24 (your local network)
  - 192.168.100.0/24
- Purpose:
  - Internet access
  - Nginx proxy for WordPress

### VM B
- IP Addresses:
  - 192.168.101.0/24
  - 192.168.100.0/24
- Purpose:
  - No Intenet access
  - Mysql on 101 network

### VM C
- IP Addresses:
  - 192.168.100.0/24
  - 192.168.102.0/24
- Purpose:
  - Internet access
  - Apache with WordPress 102 network

## Insctructions

### Prerequisites
- Install [Vagrant](https://www.vagrantup.com/) on your local machine.
### Getting Started
1. Clone this repository to your local machine.
2. Change directory to the cloned repository to Vagrant_linux_networking
3. ```
   vagrant up
   ```
### Configuration files
- Netplan configurations
  - For Virtual machine R (this is "router")
```
  ---
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 172.16.0.1/24
    enp0s9:
      dhcp4: true  
```
  - For Virtual Machine A
  - For Virtual Machine B
  - For Virtual Machine C
