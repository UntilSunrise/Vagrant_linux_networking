Vagrant.configure("2") do |config|
  # VM R (router for VM A,B,C)
   config.vm.define "VM R" do |r|
     r.vm.box = "ubuntu/focal64"
     r.vm.network "private_network", ip: "172.16.0.1", netmask: "255.255.255.0"
     r.vm.network "public_network", bridge: "eth0"
    
     r.vm.provision "shell", run: "always", inline:
      "ip route add 192.168.100.0/24 via 172.16.0.2
       ip route add 192.168.101.0/24 via 172.16.0.2
       ip route add 192.168.102.0/24 via 172.16.0.2"  
   end
  
   config.vm.define "VM A" do |a|
    a.vm.box = "ubuntu/focal64"
    a.vm.network "private_network", ip: "172.16.0.2", netmask: "255.255.255.0"
    a.vm.network "private_network", ip: "192.168.100.1", netmask: "255.255.255.0"
    a.vm.network "public_network", auto_config: false
  
    a.vm.provision "shell", run: "always", inline:
     "ip route add default via 172.16.0.1
      ip route add 192.168.101.0/24 via 192.168.100.2
      ip route add 192.168.102.0/24 via 192.168.100.3
      echo 'nameserver 8.8.8.8' | sudo tee /etc/resolv.conf
      echo 'nameserver 8.8.4.4' | sudo tee -a /etc/resolv.conf"  
   end
  
   config.vm.define "VM B" do |b|
    b.vm.box = "ubuntu/focal64"
    b.vm.network "private_network", ip: "192.168.100.2", netmask: "255.255.255.0"
    b.vm.network "private_network", ip: "192.168.101.1", netmask: "255.255.255.0"
    b.vm.network "public_network", auto_config: false
  
    b.vm.provision "shell", run: "always", inline:
     "ip route add 192.168.102.0/24 via 192.168.100.3"  
   end
  
   config.vm.define "VM C" do |c|
    c.vm.box = "ubuntu/focal64"
    c.vm.network "private_network", ip: "192.168.100.3", netmask: "255.255.255.0"
    c.vm.network "private_network", ip: "192.168.102.1", netmask: "255.255.255.0"
    c.vm.network "public_network", auto_config: false
  
    c.vm.provision "shell", run: "always", inline:
     "ip route add 192.168.101.0/24 via 192.168.100.2
      ip route add 0.0.0.0/0 via 192.168.100.1
      echo 'nameserver 8.8.8.8' | sudo tee /etc/resolv.conf
      echo 'nameserver 8.8.4.4' | sudo tee -a /etc/resolv.conf"
   end
  end
