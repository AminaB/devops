# install vm virtualbox
- install on it ubuntu and centos
- you can use vagrant to automate installation and provisioning VM in VM virtualbox
  - go to vagrantup.com to see vagrant boxes (os)
        vagrant init centos : doc to https://portal.cloud.hashicorp.com/vagrant/discover/eurolinux-vagrant/centos-stream-9 : it will create vagrant file
        vagrant box-list    
        vagrant status
        vagrant ssh : to ssh into it
        vagrant reload : to reboot
        vagrant destroy : to delete
        vagrant up : create vm again
  
    - install ubuntu using vagrant
      vagrant init ubuntu.......
      vagrant up
      vagrant halt : stop 


# vagrant vs terraform : both projects from HashiCorp.
- Vagrant is for development environments. Terraform is for more general infrastructure management.
- vagrant provisioner : shell. config.vm.provision "shell" (in vagrant file)

# Website setup using vagrant provisioning
## centos : http server & html template
     - vagrant init centos...., change the ip address and active public network and vb.memory in vigrant file
     - vagrant ssh, and change the hostname to "finance" : vi /etc/hostname, logout and login again
     - yum install httpd wget vim unzip zip -Y
     - systemctl start httpd
     - systemctl enable httpd
     - ip addr show: and connect with browser, you will see http home page    
     - add index.htm into : /var/wwww/html/
     - restart the service and test
     - download free template here : https://www.tooplate.com/view/2135-mini-finance
     - unzip name, cd into unzip ,  cp -r * /var/www/html, and restart httpd, and test
     - check firewall, systemctl status firewall (stop and disable it)
## centos iac: http server & html template, into vagrant file
  `config.vm.provision "shell", inline: <<-SHELL
     yum install httpd wget unzip -y
	 systemctl start httpd
	 systemctl enable httpd
	 cd /tmp/
	 wget https://www.tooplate.com/zip-templates/2119_gymso_fitness.zip
	 unzip -o 2119_gymso_fitness.zip
	 cp -r 2119_gymso_fitness/* /var/www/html/
	 systemctl restart httpd
   SHELL`
- vagrant up
## ubuntu : Lamp, and wordpress template, 
    - vagrant init ubuntu20....change the ip address and active public network, and vb.memory
    - vagrant ssh, and change the hostname to "wordpress" : vi /etc/hostname, logout and login again
    - read and install everything : https://ubuntu.com/tutorials/install-and-configure-wordpress#1-overview
    - or do it with bootstrapping into vagrantfile (iac)
## ubuntu iac: Lamp, and wordpress template, into vagrant file

  ` config.vm.provision "shell", inline: <<-SHELL
    sudo apt update
    sudo apt install apache2 \
    ghostscript \
    libapache2-mod-php \
    mysql-server \
    php \
    php-bcmath \
    php-curl \
    php-imagick \
    php-intl \
    php-json \
    php-mbstring \
    php-mysql \
    php-xml \
    php-zip -y`

    `sudo mkdir -p /srv/www
    sudo chown www-data: /srv/www
    curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www`

    `cat > /etc/apache2/sites-available/wordpress.conf <<EOF
    <VirtualHost *:80>
    DocumentRoot /srv/www/wordpress
    <Directory /srv/www/wordpress>
    Options FollowSymLinks
    AllowOverride Limit Options FileInfo
    DirectoryIndex index.php
    Require all granted
    </Directory>
    <Directory /srv/www/wordpress/wp-content>
    Options FollowSymLinks
    Require all granted
    </Directory>
    </VirtualHost>
    EOF`

    `sudo a2ensite wordpress
    sudo a2enmod rewrite
    sudo a2dissite 000-default`

     `mysql -u root -e 'CREATE DATABASE wordpress;'
     mysql -u root -e 'CREATE USER wordpress@localhost IDENTIFIED BY "admin123";'
     mysql -u root -e 'GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost;'   
     mysql -u root -e 'FLUSH PRIVILEGES;'`
     
     `sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
     sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
     sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
     sudo -u www-data sed -i 's/password_here/admin123/' /srv/www/wordpress/wp-config.php`
     
     `systemctl restart mysql
     systemctl restart apache2`

     `SHELL`
- vagrant up 

