# Networking component
- two or more devices
- cables as link or wireless
- network interface card (nic) in each device
- switches : to connect networks interfaces together 
- router : to connect networks together 
- OS

# model OSI (open system interconnection)
- rules : how component communicate 
- setup standard to communicate each other
- communication model
- 7 layers : app, presentation, session, transport, network, data link,,physical
- each layer provide something to the next 

# network classification by geography
- LAN : local area network :
- WAN
- MAN
- CAN

# private ip range 
- class A : 10.0.0.0 – 10.255.255.255
- class B : 172.16.0.0 – 172.31.255.255
- Class C : 192.168.0.0 – 192.168.255.255
- ips in between are public used by internet

# protocols 
- Layer 4 trasport : TCP,  UDP
- TCP : reliable protocol, reliability. FTP, HTTP, HTTPS
- UDP : ureliable protocol, faster. speed matter more that reliability. DSN, DHCP, TFTP, ARP, RARP

# networking command
- ssh web01
- sudo -i
- ifconfig or ip addr show: see active network interface, their names and ips
- vi /etc/hosts : to give name to ip addr
- ping db01
- tracert www.ggogle.com : send 3 packet to google ans show all the hubs it went through to reach google
- netstat -antp : to see all the tcp open port
- nmap localhost : show open port
- nmap web01
- dig www.google.com : show the dns server and ip
- nslookup or dig
- route -n : show gateways
- arp
- mtr google.com : to show package lost
- telnet @ip