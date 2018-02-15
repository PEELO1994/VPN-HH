# Projektin eteneminen 

## Projektin aloitus

14.2.2018 

Pidimme ohjauskokouksen, jossa päivitetty projektisuunitelmamme hyväksyttiin. Saimme käyttöön 2 tietokonetta ja 2 siirtokovalevyä. Näitä laitteta hyödyntäen tulemme pystyttämään VPN yhteyden. Pystymme kokeilemaan samanaikaisesti kahta eri ratkaisua 2 laitteen ja levyn johdosta. Ensimmäiseksi päätimme lähteä kokeilemaan OpenVPN pystytystä omille läppäreillemme ilman paineta, jotta saisimme hieman käsitystä kuinka Ubuntu Server toimii yms. Lähdemme asentamaan lopullista ratkaisua vasta myöhemmin, kun olemme haalineet enemmän tietotaitoa.

Luotiin USB- asennus tikku Ubuntu Server 16.04.3 LTS- versiolla. Asennettiin Ubuntu kannettavalle tietokoneelle tikun avulla. Software Selection kohdassa valitsimme “Basic Ubuntu Server”, “DNS Server” ja “Basic Utilities” vaihtoehdot. OpenSSH ei asennettu koska emme tarvitse sitä testiympäristössä. Asennus ei onnistunut, koneen näyttö oli mustana, eikä komentoja pystynyt antamaan. Asensimme uudestaan alusta, asennus ei onnistunut taaskaan. Tällä kertaan valittiin “install ubuntu server”, joka ei onnistunut. Tarkoitus oli valita vaihtoehto “install”. Kolmas asennus onnistui. Asennettiin samalla desktop, jolloin voidaan graafisen käyttöliittymän kautta testata openVPN: n käyttöä. Asennettiin openVPN. 

“Sudo apt-get update”
“Sudo apt-get install openvpn” 

Seuraavaksi haluttiin mahdollistaa “network bridging”, eli mahdollistaa lanien yhdistäminen. 
Sudo apt-get install bridge-utils
Tämän jälkeen meidän tuli muokata tiedostoa /etc/network/interfaces

Lisäsimme tiedostoon verkon tiedot (IP, Netmask, Broadcast, network, gateway), jonka jälkeen uudelleen käynnistimme palvelun
sudo /etc/init.d/networking restart
Saimme virheilmoituksen; Failed! 
Terminaalin antamien ohjeiden mukaan saimme lisätietoa komennolla;
Systemctl status networking.service
Saimme virhekoodi ilmoituksen joka näytti tältä:

● networking.service - Raise network interfaces
   Loaded: loaded (/lib/systemd/system/networking.service; enabled; vendor preset: enabled)
  Drop-In: /run/systemd/generator/networking.service.d
       	└─50-insserv.conf-$network.conf
   Active: failed (Result: exit-code) since Wed 2018-02-14 16:27:46 EET; 47s ago
 	Docs: man:interfaces(5)
  Process: 4063 ExecStop=/sbin/ifdown -a --read-environment --exclude=lo (code=exited, status=0/SUCCESS)
  Process: 4109 ExecStart=/sbin/ifup -a --read-environment (code=exited, status=1/FAILURE)
  Process: 4090 ExecStartPre=/bin/sh -c [ "$CONFIGURE_INTERFACES" != "no" ] && [ -n "$(ifquery --read-environment --list --exclude=lo)" ] && udevadm settle (code=exited, status=0/SUCCESS)
 Main PID: 4109 (code=exited, status=1/FAILURE)


Huomasimme että meillä on interfaces tiedostossa interfacena eth0, vaikka netstat komento näytti sen olevan wlp3s0. Vaihdoimme tiedon, mutta saimme jälleen virhe ilmoituksen failed.

Aiemmin olimme saaneet ilmoituksen, että eth0 does not exist, mutta nyt wlp3s0 löytyi. Nyt ongelmana oli seuraavanlainen teksti;
	Can’t add wlp3s0 to bridge br0: operation not supported

Motivaatiomme alkoi laskea, kun emme päässeet eteenpäin. Huomasimme, että on olemassa ohjeita mitkä olivat lähempänä meidän ohjelmisto versioita, joten päätimme jatkaa seuraavalla kerralla ja aloittaa alusta. Olimme siis katsoneet ohjeita, kuinka asentaa OpenVPN Ubuntulle, mutta kyseessä on Ubuntu Server, eli joitain eroja löytyy. 
