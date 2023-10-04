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
3. In CMD go to directory with Vagrantfile and exec command:
   ```
   vagrant up
   ```
   Use number 1 when prompted by the system. This is necessary to configure the network interface.
4. Go to machine R to register MASQUERADE, as well as activate packet routing and ping machines A, B, C.
   ```
   sudo iptables -t nat -A POSTROUTING -o enp0s3 -s 172.16.0.0/24 -j MASQUERADE
   echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
   ```
5. Go to machine A and exec this commands:
   - This need for activate packet routing and del default options from routing tables
   ```
   echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
   sudo ip route del default via 10.0.2.2
   ping 8.8.8.8
   ```
   - Install nginx and setting up proxy
   ```
   sudo apt-get update
   sudo apt-get install -y nginx
   sudo service nginx start
   cd /etc/nginx/sites-available/
   sudo touch mysite
   sudo vi mysite
   ```
   Add this server settings to conf file
   ```
   server {
            listen 80;
            server_name mysite.local;
            access_log /var/log/nginx/mysite-access.log;
            error_log /var/log/nginx/mysite-error.log;
    
            location / {
                proxy_pass http://192.168.100.3;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
            }
   }
   ```
   Check nginx conf and use next steps
   ```
   sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```
7. 
### Configuration files
- Netplan configurations
  - For Virtual machine R (this is "router")
    - netplan machine R
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
