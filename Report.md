# Project report


## First time installation
## 14.2.2018

Our updated project plan was approved in this week's group meating. Two computers and two hard drives were allocated to us as a result. We will be installing and configuring a VPN on these devices. We should be able to test out two different approaches with the two devices given to us. To kick off the project we installed OpenVPN and Ubuntu Server on our laptops to try them out before actually installing them onto a hard drive located in a desktop. We will be installing and configuring Ubuntu Server and OpenVPN on a desktop after we've had some practice with them in a safe environment.

After creating a live usb stick with Ubuntu Server 16.4.03 LTS we proceeded to install it on a laptop. The options "Basic Ubuntu Server", "DNS Server" and "Basic Utilities" were selected in the Software Selection section. We didn't install OpenSSH as it's not of any use to us in the trial phase. The installation failed with only a black screen as a result and no input from the user was registered by the computer. The second installation failed as well as we chose the option "Install Ubuntu Server" when we were supposed to simply select "Install". Our third attempt was successful. We installed a GUI as well for ease of access as it was our first time using Ubuntu Server although one should never be installed on a server operating system. After the Ubuntu Server installation was complete we installed OpenVPN with the following command:

"sudo apt-get update" 
"sudo apt-get install openvpn"

We wanted to enable "Network Bridging" so we also installed the following:

"sudo apt-get install bridge-utils"

After which we modified the file in the path: /etc/network/interfaces
We modified the file by filling in the following information: IP, Netmask, Broadcast, Network, Gateway. After modifying the file we restarted the service:

"sudo /etc/init.d/networking restart"

The restart failed so we used the command:

"systemctl status networking.service"

The end result was the following:

● networking.service - Raise network interfaces Loaded: loaded (/lib/systemd/system/networking.service; enabled; vendor preset: enabled) Drop-In: /run/systemd/generator/networking.service.d └─50-insserv.conf-$network.conf Active: failed (Result: exit-code) since Wed 2018-02-14 16:27:46 EET; 47s ago Docs: man:interfaces(5) Process: 4063 ExecStop=/sbin/ifdown -a --read-environment --exclude=lo (code=exited, status=0/SUCCESS) Process: 4109 ExecStart=/sbin/ifup -a --read-environment (code=exited, status=1/FAILURE) Process: 4090 ExecStartPre=/bin/sh -c [ "$CONFIGURE_INTERFACES" != "no" ] && [ -n "$(ifquery --read-environment --list --exclude=lo)" ] && udevadm settle (code=exited, status=0/SUCCESS) Main PID: 4109 (code=exited, status=1/FAILURE)  

The interface id we added to the file /etc/network/interfaces was faulty. We replaced the old interface id (eth0) with wlp3s0 which was acquired with the "netstat" command. The restart once again failed but for a different reason. This time the error message read "Can’t add wlp3s0 to bridge br0: operation not supported".

We noticed that we were following instructions aimed at installing OpenVPN on Ubuntu instead of Ubuntu Server which is why we decided to start over following a different set of instructions.     

## Second installation attempt
## 22.2.2018




