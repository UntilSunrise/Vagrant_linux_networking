# Vagrant_linux_networking
## Task conditions
1) Run three virtual machines with the Ubuntu OS using Vagrant
2) Each machine must have two network interfaces with static addresses from defined networks
VM A
172.16.0.0/24 (your local network)
192.168.100.0/24
Internet access
Nginx proxy for WordPress

VM B
192.168.101.0/24
192.168.100.0/24
No Intenet access
Mysql on 101 network

VM C
192.168.100.0/24
192.168.102.0/24
Internet access
Apache with WordPress 102 network
4) Fulfill the following conditions for virtual machines
You can curl WordPress by mysyte.local (added to /etc/hosts)
from IP in 172.16.0.0/24 (your local PC) network
