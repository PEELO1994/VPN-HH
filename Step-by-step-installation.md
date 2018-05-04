# OpenVPN installation and configuration tutorial 

## Some basic hardware and software that we used on this project:
	-Ubuntu 16.04
	-OpenVPN 
	-HP Compaq 8200 Elite Convertible Minitower
	-WinSCP
	

## Requirements before getting started: 

-Ubuntu 16.04 LTS server installed 

Quick note: If you try to install ubuntu 16.04 from USB-stick and the installation gives an error message something about it trying to install updates from cd-rom. We managed to find one solution to fix that error in case that happens.
So during installation if error occurs, these are the commands that you need:

!!!In case you get permission denied in some commands just add sudo in front!!!

	$ALT+F2 (Switches view to terminal screen)
	$cd /target/etc/apt
	$cp sources.list.apt-install sources.list 
	$nano sources.list (Here you need to add comment to line which indicates something about cd-rom )
	$chroot /target apt-get update 
	$chroot /target apt-get upgrade
	$ALT+F1 (Switches back to installation screen)

Now you just follow installation normally again and hopefully this works for you.


## The installation of OpenVPN and creation of the RSA keys:


After you have installed server of your choice in our case ubuntu.16.04 lts, you will start by installing OpenVPN packets that you need on your server computer.

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

    $./build-key-server VPNSERVER (To create a key)

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

    $sudo cp ca.crt VPNSERVER.crt VPNSERVER.key ta.key dh2048.pem /etc/openvpn (To copy the files onto the OpenVPN folder)
    
And after this we needed to copy and unzip OpenVPN configuration file into configuration directory so we could use it as a basis for our setup.

    $gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/VPNSERVER.conf.gz | sudo tee /etc/openvpn/VPNSERVER.conf (To copy and unzip the sample config)
    
## Adjusting the OpenVPN Configuration

First we needed to modify the server configuration file:
 
 	$sudo nano /etc/openvpn/VPNSERVER.conf
	
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

    $sudo systemctl start openvpn@VPNSERVER

    $sudo systemctl status openvpn@VPNSERVER (check whether the service restarted properly)
    
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

    $sudo systemctl enable openvpn@VPNSERVER ( if everything went well,this command will start the service automaticly when system boots)

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
    
    
## Client Installation tutorials for different operating systems

## Windows

!!!OpenVPN needs administrative privileges to install!!!

If you are using windows based OS as a client computer, first you need to install OpenVPN Client GUI application from here: https://openvpn.net/index.php/open-source/downloads.html
There you will choose appropriate installer version of your windows.

After installation you need to copy your .ovpn client file to:
	
	$C:\Program Files\OpenVPN\config
	
Quick tip: Easiest way to transfer your client.ovpn file from your linux based OpenVPN server to windows is to use WinSCP software, atleast that is what we found out to be most convenient way in our situation.  

## LINUX 

If you are using ubuntu or debian based OS you can install OpenVPN package through terminal:

	$sudo apt-get update
	$sudo apt-get install openvpn
	
Then make sure your distribution includes /etc/openvpn/update-revolv.conf script:

	$ls /etc/openvpn
	
	Output
	update-resolv-conf

Next you need to edit your client configuration file:

	nano client1.ovpn (or what ever your client files name is)

There you need to uncomment following lines:

	script-security 2
	up /etc/openvpn/update-resolv-conf
	down /etc/openvpn/update-resolv-conf

Then save and close file

Now you can try to connect to your VPN with this command:

	sudo openvpn --config client1.ovpn (or again whatever your client file name is that you have transferred)

If everything is done correctly you should now connect to your server.

## Android Phone

First you need to download OpenVPN Connect application from appstore which is the official Android OpenVPN client application.
Then you need to transfer your .ovpn file to your Smartphone whatever way you find most convenient.

After installation star the OpenVPN app and from the menu choose "import" to import the profile
Then navigate to the location where you transfered your profile and select the file.
App will tell you that profile was imported.

Then you just simply tap connect button and if everything is done correctly you should be able to connect to your OpenVPN server. 

