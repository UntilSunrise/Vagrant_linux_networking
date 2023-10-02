Vagrant.configure("2") do |config|
# VM R (router for VM A,B,C)
 config.vm.define "VM R" do |r|
   r.vm.box = "ubuntu/focal64"
   r.vm.network "private_network", ip: "172.16.0.1", netmask: "255.255.255.0"
   r.vm.network "public_network", bridge: "eth0"
  
   r.vm.provision "shell", run: "always", inline:
    "ip route add 192.168.100.0/24 via 172.16.0.2
     ip route add 192.168.101.0/24 via 172.16.0.2
     ip route add 192.168.102.0/24 via 172.16.0.2
     sudo netplan apply
     sudo iptables -t nat -A POSTROUTING -o enp0s3 -s 172.16.0.0/24 -j MASQUERADE
     echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
     sudo sysctl -p
     ping -c 5 google.com" 
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
    echo 'nameserver 8.8.4.4' | sudo tee -a /etc/resolv.conf
    sudo netplan apply
    echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
    sudo sysctl -p"

  a.vm.provision "shell", run: "always", inline:
   "sudo apt-get update
    sudo apt-get install -y nginx
    sudo service nginx start
    sudo bash -c 'cat > /etc/nginx/sites-available/mysite' <<EOF
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
      EOF
    
      sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
      sudo nginx -t
      sudo systemctl reload nginx" 
 end

 config.vm.define "VM B" do |b|
  b.vm.box = "ubuntu/focal64"
  b.vm.network "private_network", ip: "192.168.100.2", netmask: "255.255.255.0"
  b.vm.network "private_network", ip: "192.168.101.1", netmask: "255.255.255.0"
  b.vm.network "public_network", auto_config: false

  b.vm.provision "shell", run: "always", inline:
   "ip route add 192.168.102.0/24 via 192.168.100.3
    sudo netplan apply
    echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
    sudo sysctl -p"
  b.vm.provision "shell", run: "always", inline:
   "cp /vagrant/mysql.deb ~/
    sudo DEBIAN_FRONTEND=noninteractive dpkg -i ~/mysql.deb
    sudo apt-get install -y -f
    rm ~/mysql.deb
    sudo apt-get update
    sudo apt-get install -y mysql-server

    sudo mysql -e 'CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;'
    sudo mysql -e 'CREATE USER 'wordpress_user'@'%' IDENTIFIED WITH mysql_native_password BY '123pass';''
    sudo mysql -e 'GRANT ALL ON wordpress.* TO 'wordpress_user'@'%';'
    sudo mysql -e 'FLUSH PRIVILEGES;'"  
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
    echo 'nameserver 8.8.4.4' | sudo tee -a /etc/resolv.conf
    sudo netplan apply
    echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
    sudo sysctl -p"

  c.vm.provision "shell", run: "always", inline:
   "sudo apt-get update
    sudo apt-get install -y apache2
    sudo service apache2 start
    sudo apt-get install -y php libapache2-mod-php
    sudo apt-get install -y php-mysql
    sudo systemctl restart apache2
    sudo apt-get install -y curl
    echo '127.0.0.1 mysite.local' | sudo tee -a /etc/hosts
    sudo curl -O https://wordpress.org/latest.tar.gz
    sudo tar xzvf latest.tar.gz
    sudo cp -r wordpress/* /var/www/html/
    sudo chown -R www-data:www-data /var/www/html/
    sudo chmod -R 755 /var/www/html/
    cp -v wp-config-sample.php wp-config-sample.php.backup
    "
 end
end
