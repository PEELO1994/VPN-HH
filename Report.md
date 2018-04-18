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

    ALT+F2 (To access the command line)
    $cd /target/etc/apt (Enter the apt directory)
    $cp sources.list.apt-install sources.list (To copy the file onto "sources.list") 
    $nano sources.list (To modify the file) 
    Commented the line "cdrom" by adding a # in front of it   
    $chroot/target apt-get update
    $chroot/target apt-get upgrade
    "ALT+F1" (To continue the installation process)

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

    $./clean-all (To make sure there are no old certificates)
    $./build-ca (To build a root certificate)

Continue typing enter as the "vars" -file should supply all the answers.

    $./build-key-server server (To create a key)

Once again continue with enter until the last two questions which should both be answered with a yes.

    $./build-dh (To create the Diffie-Hellman keys. This may take a while)
    $openvpn -genkey -secret keys/ta.key (To create the HMAC -signature)

The Client's certificate

    $./build-key-pass client1 (To create a client certificate. Select the passphrase and continue with enter)

Configuring the OpenVPN service

    $sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn (To copy the files onto the OpenVPN folder)

    $gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf (To copy and unzip the sample config)

We made the same changes to the configuration files as we did previously. 

IP Forwarding

    $sudo nano /etc/sysctl.conf (To edit the file and uncomment the line "net.ipv4.ip_forward" by removing the #)
    $sudo sysctl -p (To update the session)

Firewall

    $-ip route | grep default (To find out the public interface network)
    $sudo nano /etc/ufw/before.rules (To enter the file and add the following):

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

## Installation and configuration of Ubuntu Server 16.04 and OpenVPN 
## 28.2.2018 


Today we finally started to install ubuntu server 16.04 and OpenVPN software to our main computer of this project that we will be using for rest of this time. The computer is located at our schools “server room” and the model of the computer is HP Compaq 8200 Elite Convertible MiniTower PC. We will be installing all the needed software and operating systems to our hard drive which will be plugged into our computer. 

Because we have already managed to install Ubuntu server and OpenVPN successfully earlier in our own computers this installation and configuration should go without any huge problems because we had already figured out all the main problems that can usually occur during the installation.

During our earlier installation times we had made for ourselves easy step by step instructions what to do during installation, so we used those to make this installation process to go as smooth as possible. You can go check this installation report out from our wordpress site.


We plugged our hard drive and USB stick that contained the installation disk image for ubuntu server 16.04 into our computer. We had prepared 2 different kind of installation options in case one wouldn’t work for some reason. These two were Ubuntu server installation from CD or from USB. We decided to install the ubuntu server 16.04 from our usb stick to the hard drive as we already knew how to get that work properly. After that we started to computer and began the installation process. 

Installation was going normal until we got the same error message after “Software selection” screen. The problem here was that the installer tried to install the softwares from nonexistent cd-rom and not from internet as it is supposed to. To fix this problem we just did the same thing as earlier, what you can go see from our wordpress site where installation process is reported step by step. After this one little error fix rest of the installation went down smoothly and finally the ubuntu server 16.04 was installed successfully to out main computer that we use in this project. Finally after installation we successfully tried to login to our server with the credentials we stated during installation. 

Now the next step was to install OpenVPN and RSA to our server from terminal. Which was accomplished just by basic “sudo apt-get install openvpn easy-rsa” command. 
All the server computer configuration that we did can be found written thoroughly from our wordpress.
After this we decided to test our now supposedly working server with couple of clients. We had one extra laptop with linux xubuntu installed and our android based smartphones so we tried if we could connect those to our VPN server.

Linux Xubuntu client Installation

First we installed and configured OpenVPN client configuration to xubuntu:

    sudo apt-get update


    sudo apt-get install openvpn

After installation we made sure that the installation had created the file we needed.

 	ls /etc/openvpn
 
“Update-resolve-conf” was where it was supposed to be

.
Next we had to to edit our client1.ovpn file that we had created in server computer and after transferred to our client devices. 

    nano client1.ovpn
 
There were three lines that needed to be uncommented:

	script-security 2
	up /etc/openvpn/update-resolv-conf
	down /etc/openvpn/update-resolv-conf

After that, all that was left, was try if our client computer can connect to our server: 

    sudo openvpn --config client1.ovpn

Connecting was successful 

Android client installation

Client installation to android was really simple. You just needed to download “OpenVpn connect” app from Google play store. After downloading and installation app opened and it had 3 different options how to connect to the server.
We decided to use option where you transferred the client1.ovpn file that we had created to our phones and simply just run it in the app. After the simple tutorial guide that app provided to us we just needed to push big button that stated “connect” and after that smartphone connected  to out server just fine.

----------------------------------------------------------------------------------------------------------------------------------------
14.3.2018

## First successful connections over untrusted networks

The port 1194/udp was opened for traffic so now we could access the internet from our Ubuntu Server. We could also now connect to the VPN while the device was connected  We attempted this with the “ping” command. We installed “tcp dump” to observe incoming and outgoing packets:

	$ Sudo apt-get install tcpdump

	$ sudo tcpdump -i eno1

After this the system started to monitor connections. We did not quite understand the outcome of it. Before learning how to analyze the connections we realized that we could already connect to the server from Haaga-Helia network. This meant that the ports had been opened for UDP traffic through port 1194 in the networks router. We just had not set the new public ip address of the server to the client file we were using. We had to create a new one.

## Creating a new client file  

We were able to access the internet from our Ubuntu Server but still couldn’t access our VPN while our devices were connected to the internet. So far the connection was established while connected to the school’s internal network. Next up we had to create a new client file with the proper ip address so that we can access the VPN from the internet. We created a new client file (client2) which contained the correct configurations so now we should be able to access the VPN from the outside web. We transferred the file to our laptop:

	$ Sudo scp ~/client-configs/files/client2.ovpn user@xxx.xxx.xxx.xxx:/home/user/

The transfer failed and we got the following message:

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!

Someone could be eavesdropping on you right now (man-in-the-middle attack)!

It is also possible that a host key has just been changed.

The fingerprint for the ECDSA key sent by the remote host is

SHA256:sdfghjkl.

Please contact your system administrator.

Add correct host key in /root/.ssh/known_hosts to get rid of this message.

Offending ECDSA key in /root/.ssh/known_hosts:1

 remove with:

 ssh-keygen -f “/root/.ssh/known_hosts” -R xxx.xx.xxx.x

ECDSA host key for 172.28.171.5 has changed and you have requested strict checking.

Host key verification failed.

lost connection

The problem was that the server’s cache had the information of the host’s verification information. The computer had been modified multiple times so that the verification information did not match the ones on the server’s cache.

We cleared the server cache in order to be able to connect to our laptop:

	$ Sudo ssh-keygen -f “/root/.ssh/known_hosts” -R xxx.xxx.xxx.xxx

We successfully connected the server to the laptop and attached and mounted the smartphone to the laptop. The file was transferred onto the phone successfully. We created a new profile using the client2 configuration and could now connect to our VPN using mobile data. We could now access the VPN from any network. 

## Testing our VPN

Requirements;

Client machine must have openVPN installed and possess the Client.ovpn file.

Client laptop is connected to a smartphones 4G network and cannot ping VPNservers public nor private ip address, as you might already know it also cannot establish ssh connections.

First we press the openVPN icon and choose our client file, then connect. Connection is established. Now when we were able to ping devices on the VPNserver’s network successfully. Using Putty we could also use ssh connection to connect to the server itself and do configurations on it even though we were outside its network.

The next step in the project would be to assign our clients dynamic addresses from a certain pool. 

## Possible scenario for test users to try our services

We hand out an USB-stick that includes an instruction text file, client.ovpn file, OpenVPN installation file.

The instruction file includes at least the following;

Instructions on how to install openVPN on windows/linux/android
Instructions on where to place the client.ovpn file
What you can do with the VPN
How to establish connection
Then we follow the test user as he/she attempts to use the VPN service. After successful/unsuccessful attempt we ask opinions from the user.


--------------------------------------------------------------------------------------------------------------------------------
11.4.2018

## Creating user autentication for more sure environment 

So we decided to drop idea of using LDAP intergration because we haven't found any good solution to make it work properly with our VPN server. LDAP solution is possible but because we are not permitted to use our schools AD data with its fullest and the course is heading to its end. It would be too much just sitting around and waiting to get that working and we are not even completely sure if LDAP solution that we have investigated would even work properly anyway. So we came up with other solution that we can use to login clients more securely if they are using our VPN.  
Well After we discussed more about LDAP intergration we changed our minds again and decided to give LDAP a chance anyway and see if we could pull this installation and cofiguration off.

## First time installation and configuration of LDAP

We decided to start testing the LDAP function. First we needed to create LDAP server that holds the user information. OpenVPN server will later get the information from the server and then decide whether to give access to the vpn-client. We entered the addresses of the VPNserver (LDAP) client and LDAP server. 

	nano /etc/hosts

172.28.175.5 example.example server
172.28.175.1 labravpn client


Next installation of the LDAP

	$ sudo apt-get update
	$ sudo apt-get -y install slapd ldap-utils

During the installation a window popped. It asked a password for the LDAP administrator. Enter password and the installation continues.


Reconfiguring LDAP

After installation we want to make some reconfigurations. In example. we want to set the hostname our selves.

	$ sudo dpkg-reconfigure slapd

No

Domain name = type a name of your choice

organisation name = -””-

Enter password for LDAP admin

Choose backend for LDAP = HDB

DB purge? = No

Move DB? = Yes

We got an error message stating that reconfiguration was incorrect so we had to retry.
This time we selected “no” to the question “move old database?” and got to the next question.

Allow LDAPv2? = no
After installation you want to verify LDAP

	$ sudo netstat -antup | grep -i 389


Setup LDAP base DN

Generating base.ldif file for the domain

	$ nano base.ldif
	dn: ou=People,dc=ldaptest,dc=local
	objectClass: organizationalUnit
	ou: People

	dn: ou=Group,dc=ldaptest,dc=local
	objectClass: organizationalUnit
	ou: Group

Building the directory structure
	$ ldapadd -x -W -D “cn=admin,dc=ldaptest,dc=local” -f base.ldif

Output = enter LDAP password -> adding new entry…


Adding LDAP users
We need to create an LDIF user file for a new user “ldapuser”
	$ nano ldapuser.ldif 

Paste:
	dn: uid=ldapuser,ou=People,dc=ldaptest,dc=local
	objectClass: top
	objectClass: account
	objectClass: posixAccount
	objectClass: shadowAccount
	cn: ldapuser
	uid: ldapuser
	uidNumber: 9999
	gidNumber: 100
	homeDirectory: /home/ldapuser
	loginShell: /bin/bash
	gecos: Test LdapUser
	userPassword: {crypt}x
	shadowLastChange: 17058
	shadowMin: 0
	shadowMax: 99999
	shadowWarning: 7

Next create a user with the ldapadd command,

	$ ldapadd -x -W -D "cn=admin,dc=ldaptest,dc=local" -f ldapuser.ldif

We got message “no such file or directory”. The problem was an typo on the file name. We removed the misstyped ldapuser.ldif and recreated it.

Now the output of the command above;
“Enter LDAP password:”

Adding a password for the user;

	$ ldappasswd -s password123 -W -D "cn=admin,dc=ldaptest,dc=local" -x "uid=ldapuser,ou=People,dc=ldaptest,dc=local"
 
-s specify the password for the username
-x username for which the password is changed
-D Distinguished name to authenticate to the LDAP server.
 
Verify LDAP entries
ldapsearch -x cn=ldapuser -b dc=itzgeek,dc=local
 
Enable LDAP login
Send LDAP events to log file /var/log/ldap.log
	$ sudo nano /etc/rsyslog.d/50-default.conf
add this line to the file-> local4.* /var/log/ldap.log
	$ sudo service rsyslog restart
 
When the server configuration was complete we began configuring the client. We began by installing the necessary files:
	$ sudo apt-get update
	$ sudo apt-get -y install libnss-ldap libpam-ldap ldap-utils nscd
After the installation we entered the server’s ip and port number when prompted. Next up we entered the domain name of the LDAP search base. (dc=ldaptest, dc=local). Next you will have to choose which idap version to use, we selected “3”. Next prompt will ask you whether you want the idap server to be the root Database admin of the client, to which we answed “no”. The next prompt will ask you whether you want to login to the database in order to retrieve information, we picked “no” as it’s no imperative that we login to the database at the time. 
After the initial setup we edited the following file:
	$ sudo nano /etc/nsswitch.conf
We updated the file so that it looks like so:
 
	#/etc/nsswitch.conf

	#Example configuration of GNU Name Service Switch functionality.
	#If you have the `glibc-doc-reference' and `info' packages installed, try:
	#`info libc "Name Service Switch"' for information about this file.
passwd:         compat ldap
group:             compat ldap
shadow:          compat ldap
gshadow:        files
hosts:              files dns
networks:       files
protocols:      db files
services:       db files
ethers:         db files
rpc:            db files
netgroup:       nis
 
After this we restarted the service:
$ sudo service nscd restart
When the configuration was done we attempted to login to the server using the newly created ldapuser but to no avail. 

----------------------------------------------------------------------------------------------------------------------------------------
18.4.2018

## Second installation attempt of LDAP 































Sources:

OpenVPN Access Server on Active Directory via LDAP:
https://www.allcloud.io/configure-openvpn-authentication-using-active-directory/

https://docs.openvpn.net/configuration/openvpn-access-server-on-active-directory-via-ldap/

https://forums.openvpn.net/viewtopic.php?t=13053 

https://www.youtube.com/watch?v=Xc4wKqAr3HI 

Epic how to: fetch linux credentials from Windows DC:

https://technet.microsoft.com/en-us/library/2008.12.linux.aspx

https://www.theseus.fi/bitstream/handle/10024/1099/Metsajoki_Kari.pdf?sequence=1&isAllowed=y

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/windows_integration_guide/

https://www.tecmint.com/join-ubuntu-to-active-directory-domain-member-samba-winbind


https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/configure-ldap-client-on-ubuntu-16-04-debian-8.html


Windows dc and ldap

https://serverfault.com/questions/294191/is-my-ad-already-an-ldap-server?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa


https://forums.openvpn.net/viewtopic.php?t=12965


User auth
https://www.youtube.com/watch?v=V6DGD4QRXVU 


https://openvpn.net/index.php/open-source/documentation/howto.html#start


Ubuntun ohjeita OpenVPN asennukseen linuxille
https://www.linux.com/learn/install-and-configure-openvpn-server-linux


Ubuntu server asennus ohjeita https://help.ubuntu.com/community/ServerGUI


How To Setup a VPN in Windows 10 : https://www.youtube.com/watch?v=6ZCiXx6KYtA

https://www.clearos.com/resources/documentation/clearos/content:en_us:kb_o_connecting_networks_with_openvpn

https://support.untangle.com/hc/en-us/articles/202391008-Connect-Multiple-Remote-Networks-with-OpenVPN


https://openvpn.net/index.php/open-source/documentation/howto.html 

https://youtu.be/4ykbyOEsKQE?t=332 


https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04


Static client address assigning:

https://michlstechblog.info/blog/openvpn-set-a-static-ip-address-for-a-client/

Dynamic client address assigning:

https://stackoverflow.com/questions/261427/how-can-openvpn-deal-with-both-dynamic-and-fixed-ip-addresses



