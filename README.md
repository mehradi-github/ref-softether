# Installing SoftEther
Installing SoftEther Server and SoftEther Client on Ubuntu

- [Installing SoftEther](#installing-softether)
  - [Installing SoftEther Server](#installing-softether-server)
  - [Installing SoftEther Client](#installing-softether-client)

## Installing SoftEther Server 

```sh
sudo su
apt update && apt upgrade -y

# Download from https://www.softether-download.com/en.aspx?product=softether
wget https://www.softether-download.com/files/softether/v4.41-9787-rtm-2023.03.14-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.41-9787-rtm-2023.03.14-linux-x64-64bit.tar.gz
tar xzf softether-vpnserver-v4.41-9787-rtm-2023.03.14-linux-x64-64bit.tar.gz

# Compile vpnserver
apt -y install build-essential wget make curl gcc  wget zlib1g-dev tzdata git ibreadline-dev libncurses-dev libssl-dev 
cd vpnserver
make
cd ..

# Moving vpnserver to /usr/local and change permissions
mv vpnserver /usr/local && cd /usr/local/vpnserver/

chmod 600 *
chmod 700 vpnserver vpncmd


# Starting vpnserver and set password
sudo ./vpnserver start
sudo ./vpncmd
ServerPasswordSet

# Creating vpnserver.service in /lib/systemd/system/
Ctrl+C
sudo cat >> /lib/systemd/system/vpnserver.service << EOF
[Unit]
Description=SoftEther VPN Server
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/vpnserver/vpnserver start
ExecStop=/usr/local/vpnserver/vpnserver stop
[Install]
WantedBy=multi-user.target
EOF

# Enabling IP forwarding
echo net.ipv4.ip_forward = 1 | ${SUDO} tee -a /etc/sysctl.conf

# Starting vpnserver.service 
systemctl enable vpnserver
systemctl start vpnserver

systemctl status vpnserver

# Opening necessary ports
ufw allow 443
ufw allow 500,4500/udp
ufw allow 1701
ufw allow 1194
ufw allow 5555

```

## Installing SoftEther Client

```sh
clinet : https://anuradha-15.medium.com/installation-guide-of-softether-vpn-client-on-linux-54a405a0ae2c
cd vpnclient 
make
cd .. 
mv vpnclient /usr/local/
cd /usr/local/vpnclient
chmod 600 *
chmod 700 vpn*

./vpnclient start

./vpncmd
2
# Create and enable Nic
VPN Client> NicCreate NICNAME
VPN Client> NicEnable NICNAME

# Create Account and set password
VPN Client> AccountCreate ACCNAME
VPN Client> AccountPassword ACCNAME


# Connect account
VPN Client> AccountConnect ACCNAME
# VPN Client> AccountDisconnect ACCNAME
VPN Client> AccountStatusGet ACCNAME
VPN Client> AccountList
VPN Client> ctrl+C

# Enabling IP forwarding
cat /proc/sys/net/ipv4/ip_forward
vi /etc/sysctl.conf
net.ipv4.ip_forward=1
sysctl -p

sudo apt install net-tools
sudo ifconfig
dhclient vpn_NICNAME


# View Network Routing Table 
sudo netstat -rn
ip route
ip r

ip route add $IP_VPN_PUBLIC/32 via $IP_INTERNET_BOX dev $LOCAL_INTERFACE
ip route del default
ip route add default via $IP_VPN_PRIVATE dev $VPN_INTERFACE

```
