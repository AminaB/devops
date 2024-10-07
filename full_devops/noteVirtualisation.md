install vm virtualbox
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


vagrant vs terraform : both projects from HashiCorp.
- Vagrant is for development environments. Terraform is for more general infrastructure management.
- vagrant provisioner : shell. config.vm.provision "shell" (in vagrant file)

Website setup using vagrant provisioning
- centos : http server & html template
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
- ubuntu : Lamp, and wordpress template
    - vagrant init ubuntu20....change the ip address and active public network, and vb.memory
    - vagrant ssh, and change the hostname to "wordpress" : vi /etc/hostname, logout and login again
    - read and install everything : https://ubuntu.com/tutorials/install-and-configure-wordpress#1-overview
    - or do it with bootstrapping into vagrantfile (iac)
