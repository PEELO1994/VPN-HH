# Project report


## First time installation
## 14.2.2018

Our updated project plan was approved in this week's group meating. Two computers and two hard drives were allocated to us as a result. We will be installing and configuring a VPN on these devices. We should be able to test out two different approaches with the two devices given to us. To kick off the project we installed OpenVPN and Ubuntu Server on our laptops to try them out before actually installing them onto a hard drive located in a desktop. We will be installing and configuring Ubuntu Server and OpenVPN on a desktop after we've had some practice with them in a safe environment.

After creating a live usb stick with Ubuntu Server 16.4.03 LTS we proceeded to install it on a laptop. The options "Basic Ubuntu Server", "DNS Server" and "Basic Utilities" were selected in the Software Selection section. We didn't install OpenSSH as it's not of any use to us in the trial phase. The installation failed with only a black screen as a result and no input from the user was registered by the computer. The second installation failed as well as we chose the option "Install Ubuntu Server" when we were supposed to simply select "Install". Our third attempt was successful. We installed a GUI as well for ease of access as it was our first time using Ubuntu Server although one should never be installed on a server operating system. After the Ubuntu Server installation was complete we installed OpenVPN with the following command:

$sudo apt-get update 
$sudo apt-get install openvpn

We wanted to enable "Network Bridging" so we also installed the following:

$sudo apt-get install bridge-utils

After which we modified the file in the path: /etc/network/interfaces
We modified the file by filling in the following information: IP, Netmask, Broadcast, Network, Gateway. After modifying the file we restarted the service:

$sudo /etc/init.d/networking restart

The restart failed so we used the command:

$systemctl status networking.service

The end result was the following:

● networking.service - Raise network interfaces Loaded: loaded (/lib/systemd/system/networking.service; enabled; vendor preset: enabled) Drop-In: /run/systemd/generator/networking.service.d └─50-insserv.conf-$network.conf Active: failed (Result: exit-code) since Wed 2018-02-14 16:27:46 EET; 47s ago Docs: man:interfaces(5) Process: 4063 ExecStop=/sbin/ifdown -a --read-environment --exclude=lo (code=exited, status=0/SUCCESS) Process: 4109 ExecStart=/sbin/ifup -a --read-environment (code=exited, status=1/FAILURE) Process: 4090 ExecStartPre=/bin/sh -c [ "$CONFIGURE_INTERFACES" != "no" ] && [ -n "$(ifquery --read-environment --list --exclude=lo)" ] && udevadm settle (code=exited, status=0/SUCCESS) Main PID: 4109 (code=exited, status=1/FAILURE)  

The interface id we added to the file /etc/network/interfaces was faulty. We replaced the old interface id (eth0) with wlp3s0 which was acquired with the "netstat" command. The restart once again failed but for a different reason. This time the error message read "Can’t add wlp3s0 to bridge br0: operation not supported".

We noticed that we were following instructions aimed at installing OpenVPN on Ubuntu instead of Ubuntu Server which is why we decided to start over following a different set of instructions.     

## Second installation attempt
## 22.2.2018

The second time around we installed Ubuntu Server 16.4 LTS on a Lenovo T400 laptop using a live USB. We used similar instructions as last time. The installation was successful except for a "installation step failed" error message displayed in the "Software selection" phase. We looked for a solution and it turned out the laptop was attempting to install the OS from a disc when it was supposed to be looking for the files online and use those to complete the installation. 

Commands used:

ALT+F2 -To access the command line
$cd/target/etc/apt -Enter the apt directory
$cp sources.list.apt-install sources.list -To copy the file onto "sources.list" 
$nano sources.list -To modify the file 
Commented the line "cdrom" by adding a # in front of it   
$chroot/target apt-get update
$chroot/target apt-get upgrade
"ALT+F1" -To continue the installation process

We proceeded on from "software selection" and realized that the update command prompted more options than last time. "Standard utility tools" was the only option we checked as we didn't need anything alse. After the installation was done the laptop booted up and the login prompt was displayed correctly.

The file "/etc/network/interfaces" was missing the "Primary network interface" -line so we added it to the file. DHCP was used.

#The primary network interface
auto lo
iface lo inet dhcp         

After the file was modified we restarted the service:

$sudo service networking restart

When using DHCP the DNS service also needs to be installed. We attempted installing it but ran into some unexpected trouble as some configuration files were missing altogether. We decided to install the OS all over again this time with DNS, in "Software selection", installed as well.

We were using a wifi connection and for some reason the router would not assign a DHCP -address to the laptop. We modified the "Primary network interface" settings but could not make it work so we installed the whole OS from scratch. This time the laptop was hooked up on the router with an ethernet cable. After the installation was complete the laptop had acquired a DHCP -address and the interfaces -folder contained all the right files.

OpenVPN and RSA keys

The installation of OpenVPN and creation of the RSA keys:

$sudo apt-get install openvpn-rsa
$make-cadir~/openvpn-ca
$nano vars

Fill in your own information and name the Key "server".

$./clean-all -To make sure there are no old certificates
$./build-ca -To build a root certificate

Continue typing enter as the "vars" -file should supply all the answers.

$./build-key-server server -To create a key

Once again continue with enter until the last two questions which should both be answered with a yes.

$./build-dh -To create the Diffie-Hellman keys. This should take a while.
$openvpn -genkey -secret keys/ta.key -To create the HMAC -signature

The Client's certificate

$./build-key-pass client1 -To create a client certificate. Select the passphrase and continue with enter.

Configuring the OpenVPN service

$sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn -To copy the files onto the OpenVPN folder.

$gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf -To copy and unzip the sample config

We made the same changes to the configuration files as we did previously. 

IP Forwarding

$sudo nano /etc/sysctl.conf -To edit the file and uncomment the line "net.ipv4.ip_forward" by removing the #
$sudo sysctl -p -To update the session

Firewall

$-ip route | grep default -To find out the public interface network
$sudo nano /etc/ufw/before.rules -To enter the file and add the following:

#START OPENVPN RULES
#NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
#Allow traffic from OpenVPN client to wlp11s0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o ens33 -j MASQUERADE
COMMIT
#END OPENVPN RULES

$sudo ufw allow 1194/udp -To allow port 1194
$sudo ufw disable
$sudo ufw enable

OpenVPN startup 

$sudo systemctl start openvpn@server 
$sudo systemctl status openvpn@server (check whether the service restarted properly)
$sudo systemctl enable openvpn@server (this will start the service when system boots)

Client configuration was done in the same manner as previously

Configuration generation script

We created a simple script to match our configuration to the proper certificates, keys and crypted files

$sudo nano ~/client-configs/make_config.sh (To add the following information):

#!/bin/bash

#First argument: Client identifier

KEY_DIR=~/openvpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn

Save the file and run it:

$chmod 700 ~/client-configs/make_config.sh

Generate client configurations

Create a configuration file for the client:

$cd ~/client-configs
$./make_config.sh client1

Make sure the file was created properly "client1.ovpn"

$sudo ls ~/client-configs/files

The following command will allow you to edit the file:

$sudo nano ~/client-configs/files/client1.ovpn

Connecting client device

We attempted connecting a smartphone to our VPN server. The first step is to transfer the .ovpn file we just created onto the phone as it contains all the keys and tools required to form the connection. We transferred the file onto a USB stick first as we couldn't move it directly to the phone. Eventually we managed to add the file to the phone's download folder.    

Next up we downloaded the OpenVPN Connect app from google play store. Upon starting the app we selected the .ovpn file and hit the import button. The app managed to attain the server's IP -address but the connection failed with a "Connection timed out" message.

We figured out the problem was most likely caused by the router's firewall and a closed port (1194).     



