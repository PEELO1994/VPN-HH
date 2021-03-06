# Project report


## First time installation
## 14.2.2018

Our updated project plan was approved in this week's group meating. Two computers and two hard drives were allocated to us as a result. We will be installing and configuring a VPN on these devices. We should be able to test out two different approaches with the two devices given to us. To kick off the project we installed OpenVPN and Ubuntu Server on our laptops to try them out before actually installing them onto a hard drive located in a desktop. We will be installing and configuring Ubuntu Server and OpenVPN on a desktop after we've had some practice with them in a safe environment.

After creating a live usb stick with Ubuntu Server 16.4.03 LTS we proceeded to install it on a laptop. The options "Basic Ubuntu Server", "DNS Server" and "Basic Utilities" were selected in the Software Selection section. We didn't install OpenSSH as it's not of any use to us in the trial phase. The installation failed with only a black screen as a result and no input from the user was registered by the computer. The second installation failed as well as we chose the option "Install Ubuntu Server" when we were supposed to simply select "Install". Our third attempt was successful. We installed a GUI as well for ease of access as it was our first time using Ubuntu Server although one should never be installed on a server operating system. After the Ubuntu Server installation was complete we installed OpenVPN with the following command:

    $sudo apt-get update 
    $sudo apt-get install openvpn easy-rsa
    
These are the needed software that you need on the server. 
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


## The installation of OpenVPN and creation of the RSA keys:

    $sudo apt-get install openvpn easy-rsa

OpenVPN is an TLS/SSL VPN. This means that it utilizes certificates in order to encrypt traffic between the server and clients. In order to issue trusted certificates, we will need to set up our own simple certificate authority.

To begin we copy the easy-rsa template directory into our home directory with the make-cadir command:    
    
    $make-cadir~/openvpn-ca

And now move to new directory that was just created 

	$cd ~/openvpn-ca
   	
Now we needed to configure the values our CA. We needed to edit the vars file within the directory. we needed to open that file in text editor:
    
    $nano vars

Fill in your own information and name the Key "server" in this case.

After this we were able to use the variables that we set and the easy-rsa utilities to build our certificate authority.
First we needed to to go to the right dierectory and there source the vars file we just created:

	$cd ~/openvpn-ca
	$source vars 
	
Everything seemed to go corretly and response we got was:
NOTE: If you run ./clean-all, I will be doing a rm -rf on /home/xxxx/openvpn-ca/keys

Now we needed to make sure that everything was clean in our environment by giving command:

    $./clean-all (To make sure there are no old certificates)
    
And after that we created our root CA:

    $./build-ca (To build a root certificate)

Now we needed to just continue pressing enter as the "vars" -file should supply all the answers.

Next we needed to create our server certificate and key pair by typing in:

    $./build-key-server server (To create a key)

Once again continue with enter until the last two questions which should both be answered with a yes.

Next step was to generate Diffie-Hellman keys that will be used during key exchange and also generate HMAC signature that will streghten our servers TLS integrity verification capabilities:

    $./build-dh (To create the Diffie-Hellman keys. This may take a while)
    $openvpn -genkey -secret keys/ta.key (To create the HMAC -signature)

## The Client's certificate and Key pair creation 

Now we needed to make sure that we were again in openvpn-ca directory:
	
	$cd ~/openvpn-ca
There again use command "source vars" just to make sure.
	
	$source vars
And now we needed to create password-protected set of credentials by using command:

    $./build-key-pass client1 (To create a client certificate. Select the passphrase and continue with enter)

## Configuring the OpenVPN service

Next step was to start configuring the OpenVPN service using the credentials and files we've generated before.

First step was to copy all the files we needed to the /etc/openvpn configuration directory by using following commands:

	$cd ~/openvpn-ca/keys

    $sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn (To copy the files onto the OpenVPN folder)
    
And after this we needed to copy and unzip OpenVPN configuration file into configuration directory so we could use it as a basis for our setup.

    $gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf (To copy and unzip the sample config)
    
## Adjusting the OpenVPN Configuration

First we needed to modify the server configuration file:
 
 	$sudo nano /etc/openvpn/server.conf
	
Adjustments that we needed to do were these:


We needed to find the HMAC section by looking for the tls-auth directive There we Removed the ";" to uncomment the tls-auth line. Below this we added the key-direction parameter that needed to be set to "0":

	$tls-auth ta.key 0 # This file is secret
	$key-direction 0
	
Next step was to find cruptographic cipher lines and modify those. 

	$cipher AES-128-CBC
	$auth SHA256
	
Next modification that we did was user and group settings and we removed the ";" at the beginning to uncomment those lines:

	$user nobody
	$group nogroup

and For last but not least there was optional step to Push DNS Changes to Redirect All Traffic Through the VPN so we decided to do this too. We uncommented following lines 

	$push "redirect-gateway ens18 bypass-dhcp"

and just below this we also uncommented following lines:

	$push "dhcp-option DNS 208.67.222.222"
	$push "dhcp-option DNS 208.67.220.220"
	
These modifications should assist clients in reconfiguring their DNS settings to use the VPN tunnel for as the default gateway.

## Server Networking Configuration


IP Forwarding

    $sudo nano /etc/sysctl.conf (To edit the file and uncomment the line "net.ipv4.ip_forward" by removing the #)
    $sudo sysctl -p (To update the session)

Next step we needed to do was to adjust the UFW Rules to Masquerade Client Connections:

Firewall

    $-ip route | grep default (To find out the public interface network)
    
Now that we had the interface associated with your default route, we open the /etc/ufw/before.rules file to add the rest of the relevant configuration:    
    
    $sudo nano /etc/ufw/before.rules (To enter the file and add the following):
    
Remember to replace ens18 in the -A POSTROUTING line below with the interface you found in the above command.    
    
#START OPENVPN RULES
#NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
#Allow traffic from OpenVPN client to ens18 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o ens33 -j MASQUERADE
COMMIT
#END OPENVPN RULES

Next we needed to tell UFW to allow forwarded packets by default as well.

	$sudo nano /etc/default/ufw

Inside that file we needed to find DEFAULT_FORWARD_POLICY="DENY" nd change the value to "ACCEPT"


Nxt we needed to modify the firewall itself to allow traffic to OpenVPN:

    $sudo ufw allow 1194/udp -To allow port 1194
    $sudo ufw allow OpenSSH
    $sudo ufw disable
    $sudo ufw enable
    
Now our server was suppossedly to configured correctly to handle OpenVPN traffic.

## OpenVPN startup 

Now we were able to try to start our server and if everything was cofigured correctly it should start smoothly.

    $sudo systemctl start openvpn@server 

    $sudo systemctl status openvpn@server (check whether the service restarted properly)
    
If everything went well, your output should look something that looks like this:

	labra-vpn-project@Open-VPN-server:~$ sudo systemctl status openvpn@VPNSERVER
	[sudo] password for labra-vpn-project:
	● openvpn@VPNSERVER.service - OpenVPN connection to VPNSERVER
   	Loaded: loaded (/lib/systemd/system/openvpn@.service; enabled; vendor preset:
   	Active: active (running) since Wed 2018-05-02 11:45:47 EEST; 12min ago
     	Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn23ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
  	Process: 1196 ExecStart=/usr/sbin/openvpn --daemon ovpn-%i --status /run/openv
 	Main PID: 1286 (openvpn)
   	CGroup: /system.slice/system-openvpn.slice/openvpn@VPNSERVER.service
           └─1286 /usr/sbin/openvpn --daemon ovpn-VPNSERVER --status /run/openvp

	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: 172.28.171.92:55268 Data C
	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: 172.28.171.92:55268 Data C
	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: 172.28.171.92:55268 Contro
	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: 172.28.171.92:55268 [harto
	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: harto/172.28.171.92:55268
	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: harto/172.28.171.92:55268
	May 02 11:53:33 Open-VPN-server ovpn-VPNSERVER[1286]: harto/172.28.171.92:55268
	May 02 11:53:36 Open-VPN-server ovpn-VPNSERVER[1286]: harto/172.28.171.92:55268
	May 02 11:53:36 Open-VPN-server ovpn-VPNSERVER[1286]: harto/172.28.171.92:55268
	May 02 11:53:36 Open-VPN-server ovpn-VPNSERVER[1286]: harto/172.28.171.92:55268
	lines 1-21/21 (END)...skipping...

After this we typed in this command, that will start the service automaticly when system boots:

    $sudo systemctl enable openvpn@server ( if everything went well,this command will start the service automaticly when system boots)

## Create Client Configuration Infrastructure

Next step, was to set up a system that will allow us to create client configuration files easily.

we needed to create Client Config Directory Structure in our home directory to store the files:

	$mkdir -p ~/client-configs/files
	
And after since our client configuration files will have the client keys embedded, we locked down permissions on our inner directory:

	$chmod 700 ~/client-configs/files
	
## Creating a Base Configuration

First step here was to copy our client configuration into our directory to use as our base condiguration:

	$cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf

Now we opened the file:

	$nano ~/client-configs/base.conf
	
Inside we needed to make lots of changes to match our requirements and stuff.

First, step was to locate the remote directive. This points the client to our OpenVPN server address. This should be the public IP address of your OpenVPN server. If you changed the port that the OpenVPN server is listening on, change 1194 (as we did) to the port you selected:
	
	$remote server_IP_address 1194
	
Then we made sure that the protocol matches the value that we used in the server configuration:

	$proto udp
	
Next we uncommented the user and group directives:

	$user nobody
	$group nogroup
	
Next comment ca,cert and key directives

	$#ca ca.crt
	$#cert client.crt
	$#key client.key

Next we needed to match cipher and auth setting that we set in the /etc/openvpn/server.conf file:

	$cipher AES-128-CBC
	$auth SHA256

Then we added directive named key-direction at the bottow of the file.(This must be set to "1" to work with server):

	$key-direction 1
	
And last modification we needed to do was add few commented lines:

	$ # script-security 2
	$ # up /etc/openvpn/update-resolv-conf
	$ # down /etc/openvpn/update-resolv-conf
	
!!!!If you run client on linux you need to uncomment these lines.!!!!!


## Creating a Configuration Generation Script

Now we created a simple script to match our configuration to the proper certificates, keys and crypted files:

    $sudo nano ~/client-configs/make_config.sh (To add the following information):

Add these lines inside the file:

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

## Generate client configurations

Create a configuration file for the client:

    $cd ~/client-configs
    $./make_config.sh client1

Make sure the file was created properly "client1.ovpn"

    $sudo ls ~/client-configs/files

The following command will allow you to edit the file in case of need:

    $sudo nano ~/client-configs/files/client1.ovpn

## Installation and connecting client device

We tested connection clients with Windows,Linux and Android phone

First we attempted connecting a smartphone to our VPN server. The first step is to transfer the .ovpn file we just created onto the phone as it contains all the keys and tools required to form the connection. We transferred the file onto a USB stick first as we couldn't move it directly to the phone. Eventually we managed to add the file to the phone's download folder.    

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

    $sudo apt-get update


    $sudo apt-get install openvpn

After installation we made sure that the installation had created the file we needed.

 	$ls /etc/openvpn
 
“Update-resolve-conf” was where it was supposed to be

.
Next we had to to edit our client1.ovpn file that we had created in server computer and after transferred to our client devices. 

    $nano client1.ovpn
 
There were three lines that needed to be uncommented:

	script-security 2
	up /etc/openvpn/update-resolv-conf
	down /etc/openvpn/update-resolv-conf

After that, all that was left, was try if our client computer can connect to our server: 

    $sudo openvpn --config client1.ovpn

Connecting was successful 

Android client installation

Client installation to android was really simple. You just needed to download “OpenVpn connect” app from Google play store. After downloading and installation app opened and it had 3 different options how to connect to the server.
We decided to use option where you transferred the client1.ovpn file that we had created to our phones and simply just run it in the app. After the simple tutorial guide that app provided to us we just needed to push big button that stated “connect” and after that smartphone connected  to out server just fine.

----------------------------------------------------------------------------------------------------------------------------------------
14.3.2018

## First successful connections over untrusted networks

The port 1194/udp was opened for traffic so now we could access the internet from our Ubuntu Server. We could also now connect to the VPN while the device was connected  We attempted this with the “ping” command. We installed “tcp dump” to observe incoming and outgoing packets:

	$Sudo apt-get install tcpdump

	$sudo tcpdump -i eno1

After this the system started to monitor connections. We did not quite understand the outcome of it. Before learning how to analyze the connections we realized that we could already connect to the server from Haaga-Helia network. This meant that the ports had been opened for UDP traffic through port 1194 in the networks router. We just had not set the new public ip address of the server to the client file we were using. We had to create a new one.

## Creating a new client file  

We were able to access the internet from our Ubuntu Server but still couldn’t access our VPN while our devices were connected to the internet. So far the connection was established while connected to the school’s internal network. Next up we had to create a new client file with the proper ip address so that we can access the VPN from the internet. We created a new client file (client2) which contained the correct configurations so now we should be able to access the VPN from the outside web. We transferred the file to our laptop:

	$Sudo scp ~/client-configs/files/client2.ovpn user@xxx.xxx.xxx.xxx:/home/user/

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

	$Sudo ssh-keygen -f “/root/.ssh/known_hosts” -R xxx.xxx.xxx.xxx

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

	$nano /etc/hosts

172.28.175.5 example.example server
172.28.175.1 labravpn client


Next installation of the LDAP

	$sudo apt-get update
	$sudo apt-get -y install slapd ldap-utils

During the installation a window popped. It asked a password for the LDAP administrator. Enter password and the installation continues.


Reconfiguring LDAP

After installation we want to make some reconfigurations. In example. we want to set the hostname our selves.

	$sudo dpkg-reconfigure slapd

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

	$sudo netstat -antup | grep -i 389


Setup LDAP base DN

Generating base.ldif file for the domain

	$nano base.ldif
	dn: ou=People,dc=ldaptest,dc=local
	objectClass: organizationalUnit
	ou: People

	dn: ou=Group,dc=ldaptest,dc=local
	objectClass: organizationalUnit
	ou: Group

Building the directory structure
	
	$ldapadd -x -W -D “cn=admin,dc=ldaptest,dc=local” -f base.ldif

Output = enter LDAP password -> adding new entry…


Adding LDAP users
We need to create an LDIF user file for a new user “ldapuser”
	
	$nano ldapuser.ldif 

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

	$ldapadd -x -W -D "cn=admin,dc=ldaptest,dc=local" -f ldapuser.ldif

We got message “no such file or directory”. The problem was an typo on the file name. We removed the misstyped ldapuser.ldif and recreated it.

Now the output of the command above;
“Enter LDAP password:”

Adding a password for the user;

	$ldappasswd -s password123 -W -D "cn=admin,dc=ldaptest,dc=local" -x "uid=ldapuser,ou=People,dc=ldaptest,dc=local"
 
-s specify the password for the username
-x username for which the password is changed
-D Distinguished name to authenticate to the LDAP server.
 
Verify LDAP entries
ldapsearch -x cn=ldapuser -b dc=itzgeek,dc=local
 
Enable LDAP login
Send LDAP events to log file /var/log/ldap.log

	$sudo nano /etc/rsyslog.d/50-default.conf
	
add this line to the file-> local4.* /var/log/ldap.log

	$sudo service rsyslog restart
	
When the server configuration was complete we began configuring the client. We began by installing the necessary files:

	$sudo apt-get update
	$sudo apt-get -y install libnss-ldap libpam-ldap ldap-utils nscd
	
After the installation we entered the server’s ip and port number when prompted. Next up we entered the domain name of the LDAP search base. (dc=ldaptest, dc=local). Next you will have to choose which idap version to use, we selected “3”. Next prompt will ask you whether you want the idap server to be the root Database admin of the client, to which we answed “no”. The next prompt will ask you whether you want to login to the database in order to retrieve information, we picked “no” as it’s no imperative that we login to the database at the time. 
After the initial setup we edited the following file:

	$sudo nano /etc/nsswitch.conf
	
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

We started working on LDAP from scratch and everything else is basically the same except the following parts:   

The only changes we made this time where in the /etc/hosts file. We also replaced every dc entry, as shown here: dc=ldaptest replaced with dc=ilari.

This is our /etc/hosts file this time:

	172.28.175.5  server
	172.28.175.6  client

This is our new base.ldif file:

	$nano base.ldif
	dn: ou=People,dc=ilari,dc=local
	objectClass: organizationalUnit
	ou: People

	dn: ou=Group,dc=ilari,dc=local
	objectClass: organizationalUnit
	ou: Group

We verified the LDAP:

	ilari@PEELO:~$ sudo netstat -antup | grep -i 389
	tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      5212/slapd      
	tcp6       0      0 :::389                  :::*                    LISTEN      5212/slapd      


	ilari@PEELO:~$ nano base.ldif

	ilari@PEELO:~$ ldapadd -x -W -D "cn=admin,dc=ilari,dc=local" -f base.ldif
	Enter LDAP Password: 
	adding new entry "ou=People,dc=ilari,dc=local"

	adding new entry "ou=Group,dc=ilari,dc=local"

	ilari@PEELO:~$ nano ldapuser.ldif
	ilari@PEELO:~$ ldapadd -x -W -D "cn=admin,dc=ilari,dc=local" -f ldapuser.ldif


For some odd reason the command we needed to use didn't work properly and we received error message:

	ldappasswd -s password -W -D "cn=admin,dc=ilari,dc=local" -x "uid=ldapuser,ou=people,dc=ilari,dc=local"
	ldappasswd: option requires an argument -- 's'
	ldappasswd: unrecognized option -s
	
Apparently it had something to do with our first password that we tried to use. After we changed the password to something else command worked right and we managed to move to the next step.

Afterwards we verified the LDAP entry:

	$ldapsearch -x cn=ldapuser -b dc=ilari,dc=local
	
The result:
	
	# extended LDIF
	#
	# LDAPv3
	# base <dc=ilari,dc=local> with scope subtree
	# filter: cn=ldapuser
	# requesting: ALL
	#

	# ldapuser, People, itzgeek.local
	dn: uid=ldapuser,ou=People,dc=itzgeek,dc=local
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
	shadowLastChange: 17058
	shadowMin: 0
	shadowMax: 99999
	shadowWarning: 7

	# search result
	search: 2
	result: 0 Success

	# numResponses: 2
	# numEntries: 1
	
---------------------------------------------------------------------------------------------------------------------------------------

20.4.2018

## Windows server 2016 Virtual machine installation using virtualbox 

We are going to create a virtual machine and install Windows Server 2016 on it. This server is going to work as the LDAP server for our OpenVPN project. The idea is that when a user tries to establish a vpn connection he/she needs to enter his/her user credentials that are already present in a database. Then our vpn-server will check the credentials from the LDAP-server and decide whether they can be accepted or not.

We used Oracle VirtualBox to create the VM. The main purpose of this is to test actual conditions so nothing specific was needed from the VM. Microsoft provides ISO images for evaluation for free on their website. We downloaded the 2016-ISO and started our VM with that media. We installed the server with GUI to make things simpler at this stage.

## Setting up LDAPS on Windows server 2016
## Setup LDAP using AD LDS

We followed the instructions in the provided link above and used all the default settings unless mentioned otherwise.
First we needed to setup AD LDS role which was straight forward installation wizard where you used all the default options that wizard suggested. 
Followed by that we created an AD LDS instance again using the setup Wizard provided by windows Server manager. We again used all the default settings the wizard suggested except in “importing LDIF files” where we needed to choose every suggested file in a list like this.

Picture
https://github.com/PEELO1994/VPN-HH/blob/master/kuva34545.png?raw=true


After that we needed to open to ADSI Edit and from there try to connect AD LDS instance to CONTOSO. There we opened connection setting and filled the values that we had chosen before. Connection was successfully connected and we should now be able to browse the directory “CN=MRS,DC=CONTOSO,DC=COM”.  
 
Next step was go to Active Directory Certificate Services to create a certificate to be used for LDAPS. This process required us to add a new role and again we used all the default settings that wizard suggested to use. We needed to restart computer here to make the configurations valid.
After this we had to open “AD CS configuration” which was found at “AD CS page”.
Then we needed to create certificate for our server. We did this according to the instructions above. REMEMBER to use your own credentials etc.
After a certificate was generated we needed to verify that our certificate was present. This was done through “manage computer certificates” and under personal certificates we were able to view our certificate that we created.

Picture
https://github.com/PEELO1994/VPN-HH/blob/master/kuva456t567.png?raw=true

Now we needed to ensure host machine account has access to the private key. Using the Certutil utility, find the Unique Container Name. Open Command Prompt in Administrator mode and run the following command: certutil -verifystore MY

Picture
https://github.com/PEELO1994/VPN-HH/blob/master/kvua567.png?raw=true


We found our private key from there as you can see from picture above.


Then we needed to go to the following location C:\ProgramData\Microsoft\Crypto\Keys\<UniqueContainerName>
ProgramData was a hidden folder so we needed to click “View” and from there enable “Hidden items” to get ProgramData to become visible. 
After we found the right file we right clicked the file and clicked properties.
There we clicked Security tab --> edit --> add new group named “NETWORK SERVICE” and add read permissions to that group. 

Picture
https://github.com/PEELO1994/VPN-HH/blob/master/kvua677.jpg?raw=true


Next step was to import the certificate into the JRE key store since our certificate “CN=VPNPOJATLDAP” is not signed by any trusted certificate authority which is configured in your JRE keystore. In order to import this certificate using the keytool utility, we first had to export this cert as a .CER from the machine certificate store:
We Clicked Start --> Search “Manage Computer Certificates” and opened it. 
We opened “personal”, right clicked VPNPOJATLDAP cert and clicked  “Export”.

→ “Certificate Export Wizard” opened 
Then again we followed the instructions of the wizard and completed the export.
Now the certificate was supposed to be successfully exported.
Next we needed to import it to JRE Keystore using the keytool command.


  




https://blogs.msdn.microsoft.com/microsoftrservertigerteam/2017/04/10/step-by-step-guide-to-setup-ldaps-on-windows-server/ 





























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



