install vm virtualbox
- install on it ubuntu and centos
- you can use vagrant to automate installation and provisioning VM in VM virtualbox
  - go to vagrantup.com to see vagrant boxes (os)
        vagrant init centos : doc to https://portal.cloud.hashicorp.com/vagrant/discover/eurolinux-vagrant/centos-stream-9 : it will create vagrant file
        vagrant box-list    
        vagrant status
        vagrant ssh 
        vagrant reload : to reboot
        vagrant destroy : to delete
        vagrant up : create vm again
  
    - install ubuntu using vagrant
      vagrant init ubuntu.......
      vagrant up
      vagrant halt : stop 
