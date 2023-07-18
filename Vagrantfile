Vagrant.configure("2") do |config|
    # VM A
    config.vm.define "VM A" do |a|
      a.vm.box = "ubuntu/focal64"
      a.vm.network "private_network", ip: "172.16.0.10"
      a.vm.network "private_network", ip: "192.168.100.10"
      a.vm.network "public_network", bridge: "eth0"
      a.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y nginx
        sudo service nginx start
      SHELL
    end
  
    # VM B
    config.vm.define "VM B" do |b|
      b.vm.box = "ubuntu/focal64"
      b.vm.network "private_network", ip: "192.168.101.10"
      b.vm.network "private_network", ip: "192.168.100.11"
      b.vm.provision "shell", inline: <<-SHELL
        # Copy MySQL package to VM B
        cp /vagrant/mysql.deb ~/
  
        # Install MySQL from the local package
        sudo DEBIAN_FRONTEND=noninteractive dpkg -i ~/mysql.deb
        sudo apt-get install -y -f
  
        # Clean up the copied package
        rm ~/mysql.deb
      SHELL
  
      # Sync the mysql.deb package from the host machine to VM B
      b.vm.synced_folder ".", "/vagrant", type: "rsync"
    end
  
    # VM C
    config.vm.define "VM C" do |c|
      c.vm.box = "ubuntu/focal64"
      c.vm.network "private_network", ip: "192.168.100.12"
      c.vm.network "private_network", ip: "192.168.102.10"
      c.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y apache2
        sudo service apache2 start
        sudo apt-get install -y php libapache2-mod-php
        sudo systemctl restart apache2
        sudo apt-get install -y curl
        echo "127.0.0.1 mysite.local" | sudo tee -a /etc/hosts
        sudo curl -O https://wordpress.org/latest.tar.gz
        sudo tar xzvf latest.tar.gz
        sudo cp -r wordpress/* /var/www/html/
        sudo chown -R www-data:www-data /var/www/html/
        sudo chmod -R 755 /var/www/html/
      SHELL
    end
  end
  