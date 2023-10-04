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
   - Add this server settings to conf file
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
   - Check nginx conf and use next steps
   ```
   sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```
6. Go to machine B and exec commands:
   - This need for activate packet routing and del default options from routing tables:
   ```
   echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
   sudo ip route del default via 10.0.2.2
   ping 8.8.8.8
   ```
   If you have ping it's not configured correctly
   - Install mysql and create db with user for WP:
   ```
   cp /vagrant/mysql.deb ~/
   sudo DEBIAN_FRONTEND=noninteractive dpkg -i ~/mysql.deb
   sudo apt-get install -y -f
   rm ~/mysql.deb
   sudo apt-get update
   sudo apt-get install -y mysql-server
    
   sudo mysql -e "CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;"
   sudo mysql -e "CREATE USER 'wordpress_user'@'%' IDENTIFIED WITH mysql_native_password BY 'your_password';"
   sudo mysql -e "GRANT ALL ON wordpress.* TO 'wordpress_user'@'%';"
   sudo mysql -e "FLUSH PRIVILEGES;"
   ```
7. Go to machine C and exec commadns:
   - This need for activate packet routing and del default options from routing tables:
   ```
   echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
   sudo ip route del default via 10.0.2.2
   ping 8.8.8.8
   ```
   - Setting up apache and WP:
   ```
   sudo apt-get update
   sudo apt-get install -y apache2
   sudo service apache2 start
   sudo apt-get install -y php libapache2-mod-php
   sudo apt-get install -y php-mysql
   sudo systemctl restart apache2
   sudo apt-get install -y curl
   echo "127.0.0.1 mysite.local" | sudo tee -a /etc/hosts
   sudo curl -O https://wordpress.org/latest.tar.gz
   sudo tar xzvf latest.tar.gz
   sudo cp -r wordpress/* /var/www/html/
   sudo chown -R www-data:www-data /var/www/html/
   sudo chmod -R 755 /var/www/html/
   sudo cp wp-config-sample.php wp-config-sample.php.backup
   ```
   Than you need write correctly the registration files associated with the database settings and its credentials:
   ```
   sudo vi wp-config-sample.php
   ```
   And change this:
   ```
   // ** Database settings - You can get this info from your web host ** //
   /** The name of the database for WordPress */
   define( 'DB_NAME', 'wordpress' );

   /** Database username */
   define( 'DB_USER', 'wordpress_user' );

   /** Database password */
   define( 'DB_PASSWORD', 'your_password' );

   /** Database hostname */
   define( 'DB_HOST', '192.168.100.2' );
   ```
8. Congratulations. Done!
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
    - ip route show:
    ```
    
    ```
  - For Virtual Machine A
  - For Virtual Machine B
  - For Virtual Machine C
    - netplan machine C
```
      ---
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 192.168.100.3/24
    enp0s9:
      addresses:
      - 192.168.102.1/24
```

    - ip route show
```
    default via 192.168.100.1 dev enp0s8
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100
192.168.100.0/24 dev enp0s8 proto kernel scope link src 192.168.100.3
192.168.101.0/24 via 192.168.100.2 dev enp0s8
192.168.102.0/24 dev enp0s9 proto kernel scope link src 192.168.102.1
```
