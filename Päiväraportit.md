# Raportit
----------------------------------------------------------------------------------------------------------------------------------------




# Työraportti


14.2.2018 

## Serverin kokeiluasennus


Tarkoituksena on asentaa Ubuntu Server ensimmäistä kertaa kokeiluna. Luotiin USB asennustikku Ubuntu Server 16.04.3 LTS- versiolla. Asennettiin Ubuntu kannettavalle tietokoneelle tikun avulla. Software Selection kohdassa valitsimme “Basic Ubuntu Server”, “DNS Server” ja “Basic Utilities” vaihtoehdot. OpenSSH ei asennettu koska emme tarvitse sitä testiympäristössä. Asennus ei onnistunut, koneen näyttö oli mustana, eikä komentoja pystynyt antamaan. Asensimme uudestaan alusta, asennus ei onnistunut taaskaan. Tällä kertaan valittiin “install ubuntu server”, joka ei onnistunut. Tarkoitus oli valita vaihtoehto “install”. Kolmas asennus onnistui. Asennettiin samalla desktop, jolloin voidaan graafisen käyttöliittymän kautta testata openVPN: n käyttöä. Asennettiin openVPN. 

sudo apt-get update
sudo apt-get install openvpn

Seuraavaksi haluttiin mahdollistaa “network bridging”, eli mahdollistaa lanien yhdistäminen. 

sudo apt-get install bridge-utils

Tämän jälkeen meidän tuli muokata tiedostoa “/etc/network/interfaces”.

Lisäsimme tiedostoon verkon tiedot (IP, Netmask, Broadcast, network, gateway), jonka jälkeen uudelleen käynnistimme palvelun.

sudo /etc/init.d/networking restart

Saimme virheilmoituksen; Failed! 
Terminaalin antamien ohjeiden mukaan saimme lisätietoa komennolla;

systemctl status networking.service

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

Aiemmin olimme saaneet ilmoituksen, että eth0 does not exist, mutta nyt wlp3s0 löytyi. Nyt ongelmana oli seuraavanlainen teksti; Can’t add wlp3s0 to bridge br0: operation not supported.

Huomasimme, että on olemassa ohjeita mitkä olivat lähempänä meidän ohjelmistoversioita, joten päätimme jatkaa seuraavalla kerralla ja aloittaa alusta. Olimme siis katsoneet ohjeita, kuinka asentaa OpenVPN Ubuntulle, mutta kyseessä on Ubuntu Server, eli joitain eroja löytyy. 

Luotiin hakemisto:

sudo mkdir /etc/openvpn/easy-rsa/


20.2.2018

## Serverin ensimmäinen asennus


Aloitettiin laittamalla asennus Ubuntu cd:n Host palvelimeen, jossa pyöri Vmware ESXi 6.0. vSphere Client ohjelmalla. Käynnistimme yhden luomistamme virtuaalikoneista ja asensimme Ubuntun siihen.
ISO tiedostot voi helposti siirtää Hostin Datastoreen vSphere clientissä, configuration/storage/data storage alta browse datastore. Täällä voi kätevästi ladata tiedostoja Hostille. 
Boottasimme toisen virtuaalikoneen Ubuntu Server 16.04 asennusmedialla.
Install Ubuntu Server
Language	English
Location	Finland
Locales	US
Keyboard	no, valitse itse Finnish
Hostname	ubuntu server
username	lassi
encryption	no
install	guided, use all space
proxy	none
updates	automatically
software	painettiin vahingossa enteriä, ainakin ssh olisi tullut ottaa
Grub	yes

Tällä kertaa asennus näytti toimivan, niin kuin pitikin ja boottauksen jälkeen saatiin login promptin näkyviin, toisin kuin viime viikolla kun olimme tekemässä ensimmäistä kokeiluasennusta.

Seuraavaksi halusimme antaa staattisen osoitteen serverille, jotta yhteyden ottaminen on selkeämpää. Täytyi muokata tiedostoa “/etc/network/interfaces”
Tiedosto näytti tältä, kun käytössä oli dhcp:

Tältä se näytti, kun olimme laittaneet staattisen osoitteen:		   

Seuraavaksi komento 
sudo service networking restart
Yhteyden muodostaminen ei onnistunut uudella osoitteella. Oletan että SSH:ta ei ole sallittu ja sen takia emme saaneet putty:llä yhteyttä. Huomattiin tämän jälkeen että OpenSSH ei ollut asennettuna, joten asennettiin se:
sudo apt-get install openssh-server
Luotiin backup kotihakemistoon
sudo cp /etc/ssh/sshd_config/home/lassi		

Ongelmana oli nyt se, että emme onnistuneet pingaamaan testiverkon ulkopuolelta virtuaalikoneisiin. Ongelma oli virheellinen netmask. Asennettiin molempiin virtuaalikoneisiin ssh demonit, jotta voimme ottaa helposti niihin yhteyden.
Seuraavaksi asennettiin Ubuntu Serverille OpenVPN ja RSA avainten luontia varten:
sudo apt-get install openvpn easy-rsa
Sertifikaateilla salataan liikenne, joten seuraavaksi kopioitiin kotihakemistoon easy-rsa template hakemiston.
make-cadir ~/openvpn-ca
Sitten kansion sisään muokkaamaan sertifikaattia.
nano vars
-komennolla pääsee muokkaamaan vars tiedostoa, josta löytyy sertifikaatin konfiguraatiot. Täältä käytiin vaihtamassa seuraavaan kohtaan omat tiedot. Vaihdettiin myös “KEY_NAME” kohtaan nimeksi ”server”.
export KEY_COUNTRY= “FI”
export KEY_PROVINCE= “CA”
export KEY_CITY= “Espoo”
export KEY_ORG= “FORT-FUNSTON”
export KEY_EMAIL= “lassi@xxx.xxx.xxx.xxx”
export KEY_OU “MyOrganisationalUnit”

Seuraavaksi “vars” kansio tuli poluttaa eli komento ja ulostulo kuten kuvassa:

Seuraavaksi varmistetaan, että ollaan puhtaassa ympäristössä ja annetaan komento:
	./clean-all
Nyt root-sertifikaatin luonti komennolla:
	./build-ca 
Kuvasta näkyy tiedot jotka syötettiin.


21.2.2018

## Serverin toinen asennus


Sertifikaatti, avain ja kryptaus tiedostot:
Yritettiin luoda avaimia, mutta saimme seuraavan ilmoituksen;
 lassi@ubuntuserver:~$ ./build-key-server server -bash: ./build-key-server: No 	such file or directory
Olimme kotihakemistossa, joten kokeilimme samaa mutta tällä kertaa siirryimme openvpn kansion sisään. Nyt tuli seuraavaa;

Annoimme “source ./vars” komennon uudestaan, jonka jälkeen komento onnistui.
Sertifikaatin luomiseksi, täytyi painaa enteriä kysymyksiin joissa oli valmiiksi oikeat asetukset. Asetukset tulivat “vars” tiedostosta sourcen ansiosta.
Seuraavaksi loimme Diffie-Hellman salauksen. Se on salausprotokolla, jota käytetään avainten vaihdon yhteydessä.
Tämä kestää hieman, eli hae kahvia:
 ./build-dh		
Sitten HMAC allekirjoituksen luonti vahvistamaan serverin TLS:n eheyden tarkistuskykyjä.
HMAC = Hash-based message authentication code, sitä voidaan käyttää sekä tiedon eheyden varmistamiseen tai viestin autenttisuuden tarkistamiseen.
openvpn –genkey –secret keys/ta.key

### Clientin sertifikaatti

Yksinkertaisuuden nimissä, luomme clientille tarkoitetun sertifikaatin server- koneella. Näitä voi luoda niin paljon kun haluaa jos on useita clienttejä. Halusimme luoda sertifikaatille myös salasanan, joten annettiin seuraavat komennot:
source vars
./build-key-pass client1
Taas enteriä painamalla eteenpäin, koska tiedot on jo haettu “vars” -tiedostosta.

### Konfiguroidaan OpenVPN -palvelua

Seuraavaksi tulee kopioida tähän mennessä luodut tiedostot /etc/openvpn hakemistoon. 
cd /openvpn-ca/keys
sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn
Seuraavaksi kopioitiin ja purettiin OpenVPN sample konfiguraatio -tiedoston, josta on hyvä lähteä alkuun omissa asetuksissa.
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo  tee /etc/openvpn/server.conf
Sitten konfiguroimaan:
sudo nano /etc/openvpn/server.conf
“tls-auth” kohdasta poistetaan puolipiste ja lisätään alle ”key-direction 0”

Suoraan tämän alapuolelta löytyy osio “Cryptographic cipher”, josta poistimme puolipisteen “AES-128-CBC” kohdalta. AES tarjoaa vahvan kryptauksen.
Sen alapuolelle vielä rivi: “auth SHA256” (purkaa HMAC viestin).
Lopuksi vielä pari osiota alempana. Poistetaan puolipisteet kohdista ”user nobody” ja ”group nogroup”
Vaihtoehtoisia konfiguraatioita:
Ohjataan kaikki web liikenne VPN:n läpi. Otetaan puolipisteet pois kohdista:
	push ”redirect-gateway def1 bypass-dhcp”
	push “dhcp-option DNS xxx.xxx.xxx.xxx”
	push “dhcp-option DNS xxx.xxx.xxx.xxx”
Käytettäviä portteja pystyy myös vaihtamaan, mikäli sille on tarvetta, mutta tässä vaiheessa emme niitä vaihtaneet.

### IP Forwarding

Serverille täytyy mahdollistaa liikenteen ohjaus, jotta VPN:n toiminnallisuutta voidaan hyödyntää. Muokkasimme tiedostoa “/etc/sysctl.conf” postamalla risuaidan kohdan ”net.ipv4.ip_forward” edestä. Tämän jälkeen tallennus ja uusien asetusten päivitys istunnolle.
sudo nano /etc/sysctl
sudo sysctl –p

### Tulimuuri

Seuraavaksi muutimme palomuurin sääntöjä, jotta VPN pystyy naamioimaan clienttien yhteyksiä. Ensin selvitetään 
public network interface -> ip route | grep default
Sitten muokataan sääntöjä:
sudo nano /etc/ufw/before.rules
Sinne lisäsimme seuraavat rivit. Tämä lisättiin ensimmäisen sektion perään eli noin 10:lle riville. Tässä pitää muistaa lisätä juuri tarkastama interfacen nimi, jonka merkkasimme punaisella.
#START OPENVPN RULES
#NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
	# Allow traffic from OpenVPN client to wlp11s0 (change to the interface you   discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o ens33 -j MASQUERADE
COMMIT
#END OPENVPN RULES
Tämän jälkeen palomuurille täytyy kertoa, että ohjatut paketit hyväksytään defaulttina. Muutetaan ufw -tiedostosta kohtaan:
 DEFAULT_FORWARD_POLICY -> ACCEPT
sudo nano /etc/default/ufw	

### Portin avaus

OpenVPN:lle on avattava portti, jotta liikenne kulkee. 
sudo ufw allow 1194/udp
sudo ufw disable
sudo ufw enable

Käynnistä and ota käyttöön OpenVPN -palvelu

Seuraavaksi OpenVPN:n käynnistys. Tässä täytyy muistaa “conf”- tiedoston nimi, mihin olemme omat asetukset määrittäneet. 
(server on “conf” -tiedoston nimi):
sudo systemctl start openvpn@server 
(tarkista käynnistyikö palvelu, vihreällä active):
sudo systemctl status openvpn@server 
(käynnistää palvelun aina bootattuaan):
sudo systemctl enable openvpn@server 

Client Configuration Infrastructure (tehdään edelleen server- koneella)

Luodaan ensin hakemisto kotihakemistoon ja lukitaan oikeudet “files” -hakemistoon, koska siellä tulee olemaan avaimia joita ei kenenkään tarvitse päästä näkemään.
sudo mkdir –p ~/client-configs/files
chmod 700 ~/client-configs/files
Kopioidaan valmis configuration -pohja hakemistoon ja muokataan sitä:
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
nano ~/client-configs/base.conf
Ensin muokataan kohtaa “remote server_IP_address 1194”. Laitetaan serverin public IP ja oikea portti jos sitä on vaihdettu. Varmistimme myös että vieressä näkyvä asetus protokollasta on UDP.
Poistimme puolipisteet kohdista “user” ja “group”.
Alaspäin selaamalla löytyy “SSL/TLS perms”, sieltä kommentoi risuaidalla kohdat “ca”, “cert” ja “key”. Ne lisätään myöhemmin itse tiedostoon.
“Cipher” ja “Auth” laitetaan samalla tavalla, kuin aiemminkin tiedostossa “server.conf”
	cipher AES-128-CBC
	auth SHA256
Tiedoston loppuun uusi rivi -> ”key-direction 1”
Loppuun vielä kolme kommentoitua riviä;
	# script-security 2
	# up /etc/openvpn/update-resolv-conf
	# down /etc/openvpn/update-resolv-conf
Nämä otetaan käyttöön jos clientillä linux, tässä lainattu teksti näihin liittyen:
“Finally, add a few commented out lines. We want to include these with every config, but should only enable them for Linux clients that ship with a ”/etc/openvpn/update-resolv-conf” file. This script uses the resolvconf utility to update DNS information for Linux clients.
If your client is running Linux and has an ”/etc/openvpn/update-resolv-conf” file, you should uncomment these lines from the generated OpenVPN client configuration file.”


23.2.2018

## Ubuntu Server ja OpenVPN testausta


Aloitimme asentamalla Lenovon T400 läppäriin Ubuntu Server 16.04 LTS USB-tikulta. Seurasimme ohjeita jotka oli luotu virtuaaliympäristön testauksessa aikaisemmin viikolla. Asennus sujui muuten normaalisti, mutta “software selection” kohdassa saimme ilmoituksen “installation step failed!”. Googlettelimme ongelmaa ja löysimme mahdollisen ratkaisun. Ongelma oli ilmeisesti siinä, että tietokone yritti asentaa ohjelmistoja olemattomalta CD-levyltä, kun tarkoitus oli ladata tiedot internetistä.

	(pääsy komentokehotteeseen):

ALT + F2 

(siirryttiin apt hakemistoon):

cd /target/etc/apt 

(kopioitiin tiedosto “sources.list”:iin):

cp sources.list.apt-install sources.list 
nano sources.list

Kommentoitiin rivi cdrom laittamalla # rivin eteen:

chroot /target apt-get update 
chroot /target apt-get upgrade

(takaisin asennukseen):

ALT + F1 

Seuraavaksi kokeilimme uudestaan jatkaa kohdasta “software selection”. Huomasimme että update komento oli tuonut paljon lisävaihtoehtoja. Emme ottaneet mitään turhaa, ainoastaan “standard utility tools”. Asennuksen päätyttyä boottauksen jälkeen saimme login promptin näkyviin, niin kuin pitikin:

Kävimme katsomassa tiedostoa “/etc/network/interfaces” ja huomasimme, että sieltä puuttuu kokonaan kohta “primary network interface”, joten laitoimme sen sinne ja otimme käyttöön dhcp:n, koska olimme kotiympäristössä.

#The primary network interface
auto lo
iface lo inet dhcp

Tämän jälkeen restart:

sudo service networking restart

Tajusimme että, tarvitsemme DNS palvelun kun käytämme dhcp:tä, tai niin luulimme ainakin. DNS:n asennuksessa ilmeni ongelmia. Esimerkiksi conf -tiedostoja ei tullut järjestelmään vaikka niiden olisi pitänyt tulla. Päätimme asentaa koko järjestelmän alusta. Tällä kertaa otamme käyttöön “software selection” -kohdassa myös DNS palvelun.

Käytimme koko ajan wi-fi -yhteyttä ja jostain syystä reititin ei suostunut antamaan wi-fi:n kautta dhcp osoitetta. Yritimme vaihtaa “primary network interface” asetuksia, mutta emme millään saaneet osoitteita toimimaan. Päädyimme lopulta kokeilemaan asennusta kolmannen kerran kytkemällä koneen ensin kaapelilla verkkoon. Kaapelin ollessa kiinni saimme heti dhcp osoitteen niin kuin pitikin. Nyt “interfaces” kansion sisältä löytyi asennuksen jälkeen oikeat tiedot dhcp:n käytöstä.

OpenVPN ja RSA avaimet

sudo apt-get install openvpn easy-rsa

make-cadir ~/openvpn-ca

nano vars

Omat tiedot näihin export kohtiin. Myös Key name kohtaan “server”.

Olemassa olevien sertifikaattien tyhjennys:

./clean-all

Root sertifikaatti:

./build-ca

Kaikkiin kysymyksiin enter, koska “vars” -kansiosta tulee valmiiksi juuri syöttämämme tiedot.

Seuraavaksi loimme avaimen komennolla 

./build-key-server server

Taas painetaan enteriä kysymyksiin ja lopuksi yes kahteen kysymykseen.

Seuraavaksi Diffie-Hellman salausavainten vaihtoa varten komennolla:
	
	Tässä kestää hieman:

./build-dh		

HMAC allekirjoituksen luonti vahvistamaan serverin TLS:n eheyden tarkistuskykyjä:

openvpn –genkey –secret keys/ta.key

Clientin sertifikaatti

./build-key- pass client1

Passphrase, jonka jälkeen enteriä kysymyksiin.

Konfiguroidaan OpenVPN -palvelu

Kopioidaan luodut tiedot openvpn kansioon.

sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn

Seuraavaksi kopioitiin ja purettiin sample konfiguraatio

gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf

“Conf” -tiedostoon tehtiin samat muutokset, kuin aiemmassa testausdokumentissa.

### IP Forwarding

	sudo nano /etc/sysctl.conf
	
	Risuaita pois kohtasta ”net.ipv4.ip_forward”
	
	Päivitetään istunto:

sudo sysctl -p	

### Tulimuuri

Selvitetään “public network interface - ip route | grep default”

sudo nano /etc/ufw/before.rules

Lisätään seuraava:

#START OPENVPN RULES
#NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
#Allow traffic from OpenVPN client to wlp11s0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o ens33 -j MASQUERADE
COMMIT
#END OPENVPN RULES

Portin avaus:

sudo ufw allow 1194/udp

sudo ufw disable
sudo ufw enable

### OpenVPN käynnistys ja käyttöönotto

Server on konfiguraatio -tiedoston nimi:

sudo systemctl start openvpn@server

Tarkista käynnistyikö palvelu, vihreällä active:

sudo systemctl status openvpn@server

Käynnistää palvelun aina boottauksessa:

sudo systemctl enable openvpn@server 

Client konfiguraatio toteutettiin samalla tavalla, kuin edellisessä dokumentissa.

### Configuration Generation Script

Loimme yksinkertaisen skriptin, joka yhdistää konfiguraation oikeisiin sertifikaatteihin, avaimiin ja kryptaus -tiedostoihin. 

sudo nano ~/client-configs/make_config.sh

syötettiin seuraavat tiedot;

#!/bin/bash

	# First argument: Client identifier

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

Tallenna, poistu ja merkitse tiedosto ajettavaksi:

chmod 700 ~/client-configs/make_config.sh

### Generoi client konfiguraatiot 

Luodaan “client1”:lle konfiguraatio -tiedosto:

cd ~/client-configs
./make_config.sh client1

Tarkistetaan luotiinko tiedosto, pitäisi näkyä “client1.ovpn”

sudo ls ~/client-configs/files

Jos haluat muokata “files” -kansiota “client1.ovpn” tiedostoja:

sudo nano ~/client-configs/files/client1.ovpn

### Yhdistetään client laitteeseen

Kokeilimme aluksi saada älypuhelimen yhdistettyä VPN -serveriimme. Ensin meidän täytyi saada juuri luomamme .ovpn tiedosto siirrettyä puhelimeen. Tiedosto sisälsi siis tarvittavat työkalut yhteyden muodostamiseen (avaimet yms.). Siirsimme tiedostot USB:lle koska suora tiedoston siirto älypuhelimeen ei onnistunut. Saimme .ovpn tiedoston siirrettyä puhelimen downloads kansioon.

Latasimme puhelimella Androidin Google Play:stä OpenVPN Connect -sovelluksen. Sovelluksen avattua valitsimme vaihtoehdon open .ovpn file. Sovellus löysi heti tiedoston ja valittuamme sen painoimme IMPORT nappia. Tämän jälkeen sovellus tunnisti serverin tiedot, kuten IP-osoitteen johon yhteys luodaan. Kokeilimme muodostaa yhteyden, mutta se epäonnistui. Saimme ilmoituksen Connection timed out. 

Tutkimme ongelmaa netistä ja OpenVPN sovelluksen lokitiedostoista sekä kävimme läpi sovelluksen asetuksia. Olemme nyt melko varmoja, että ongelma piilee talouden reitittimen palomuuriasetuksissa. Todennäköistä on, että portti 1194 jota OpenVPN:mme käyttää on suljettu. 


28.2.2018

## Ubuntu Server 16.04 ja OpenVPN asennus ja konfigurointi


Pääsimme vihdoinkin asentamaan Ubuntu server 16.04:n ja openvpn:än projektimme lopulliselle koneelle, jossa oikea käyttö tulee tapahtumaan.
Rauta jota käytämme tässä vaiheessa on HP:n pöytäkone, johon asetamme siirto -kovalevyn johon tulemme asentamaan kaiken tarvittavan tässä vaiheessa. 
Koska olimme jo aikaisemmin asentaneet Ubuntu server 16.04 kolmelle eri testikoneelle ja saaneet Ubuntu serverin ja openVPN -palvelun toimimaan olimme valmistautuneet hyvin tätä vaihetta varten.

Käytimme asennuksessa aikaisemmin tekemiämme ohjeita jotka löytyvät täältä: https://vpnprojekti.wordpress.com/ 
https://docs.google.com/document/d/1igediP5VD2BkQAikhPsEWrMBsLqCjeY67fdopYqZ_Sg/edit 

Ubuntu server 16.04 asentamiseksi meillä oli varattuna kaksi eri vaihtoehtoa (USB-tikku, cd levy) siltä varalta jos toinen asennustapa ei olisi jostain syystä onnistunut. Aloitimme asettamalla siirto -kovalevyn sille tarkoitettuun paikkaan tietokoneessa ja päätimme aloittaa asennuksen USB -tikun kautta, koska olimme saaneet selvitettyä aiemmin miten yleisimmät asennuksessa tapahtuvat ongelmat voi helposti korjata asennuksen aikana. Asennus eteni normaalisti “software selection” kohtaan asti jonka jälkeen törmäsimme samaan ongelmaan mikä oli ollut aikaisemmissakin asennuksissa. Ongelmana oli että tietokone koitti asentaa valitsemamme ohjelmia olemattomalta cd levyltä eikä internetin kautta niin kuin oli tarkoitus. Tämän kuitenkin saimme korjattua helposti samalla tavalla kuin ennen. Tämän jälkeen Ubuntu server 16.04 loppuasennus eteni normaalisti ja onnistuimme kirjautumaan serveriin sisään antamillamme käyttäjätunnuksella ja salasanalla.

Tämän jälkeen asensimme OpenVPN -ohjelman onnistuneesti koneelle terminaalin kautta. 

Tämän jälkeen toistimme kaikki samat konfiguroinnin vaiheet kuin aikaisemmissa asennuksissa. Asettamamme konfiguraatio ja asennukset löytyvät täältä: 
https://docs.google.com/document/d/1igediP5VD2BkQAikhPsEWrMBsLqCjeY67fdopYqZ_Sg/edit 

Kun saimme kaiken server -koneen puolelta kuntoon päätimme nyt asentaa client ohjelmistot normaalille xubuntu läppärille ja android pohjaiselle älypuhelimelle. Siirsimme ennen asennusta luomamme “client1.ovpn” tiedoston tietokoneelle ja älypuhelimille joita halusimme käyttää client koneina.

### Linux asennus

Ensiksi asensimme Client konfiguraatiot Xubuntu:lle komennoilla:

sudo apt-get update
	
sudo apt-get install openvpn

Asennuksen jälkeen tarkistettiin oliko oikea tiedosto luotu komennolla:

 	ls /etc/openvpn
 
Täältä löytyi tarvittava tiedosto “update-resolve-conf”

Seuraavaksi meidän täytyi muokata “client1.ovpn” -tiedostoa client -koneella ja ottaa kolmesta tietystä kohdasta kommentti pois.

nano client1.ovpn
 
Kohdat joista täytyi ottaa kommentit pois näyttivät tältä: 

script-security 2
	up /etc/openvpn/update-resolv-conf
	down /etc/openvpn/update-resolv-conf

Tämän jälkeen tuli vielä kokeilla saako client kone yhteyden pystyttämäämme serveriin komennolla:

sudo openvpn --config client1.ovpn

Yhdistäminen onnistui normaalisti.

### Android asennus

Koska meillä sattui olemaan älypuhelimia käytössä päätimme vielä loppuun kokeilla jos saisimme otettua yhteyden myös niistä.

Ensiksi Google Play storesta täytyi ladata Android OpenVPN Connect niminen sovellus.
Tämän jälkeen käynnistimme sovelluksen. Sovelluksessa oli mahdollista ottaa yhteys kolmella eri tavalla. Valitsimme näistä kolmesta vaihtoehdosta sellaisen, missä älypuhelimeen täytyi siirtää luomamme client1.ovpn niminen tiedosto ja ajaa se sovelluksessa. Seurasimme Android OpenVPN connect sovelluksen antamia yksinkertaisia ohjeita ja saimme yhdistettyä älypuhelimen myös serveriin.

Lopetimme tältä erää tähän ja jätimme serverin pyörimään servulaan.


14.3.2018

## Luodaan uusi client tiedosto


Portti 1194/udp on nyt auki liikenteelle, mikä tarkoittaa että meidän Ubuntu Server voi nyt ottaa yhteyden internetiin. Tässä vaiheessa pystymme ottamaan salatun yhteyden VPN serveriin. Seuraavaksi asennettiin “tcp dump” -ohjelma, jotta voimme seurata ip -pakettien liikennettä.

	Sudo apt-get install tcpdump 

Seuraavalla komennolla seurasimme liikennettä:

	Sudo tpcdump -i eno1

Tässä vaiheessa saimme yhteyden VPN -server koneeseen laitteillamme mutta labraverkon ulkopuolelta ei voi edelleenkään ottaa yhteyttä. Me teimme uuden client tiedoston, ”client2.ovpn”, jossa on kaikki oikeat konfiguraatiot. Näiden avulla pitäisi pystyä ottamaan yhteyden VPN serveriin millä tahansa verkkoyhteydellä. Siirsimme tiedoston kannettavalle:

Sudo scp ~/client-configs/files/client2.ovpn ilari@xxx.xx.xxx.x:/home/ilari/

Siirto epäonnistui seuraavalla näkymällä: 
	@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:xgNfKDUVhbhU9+l+IzZCLci/xnQkH31ByrIMpZ/Sbco.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /root/.ssh/known_hosts:1
remove with:
ssh-keygen -f "/root/.ssh/known_hosts" -R xxx.xx.xxx.x
ECDSA host key for 172.28.171.5 has changed and you have requested strict checking.
Host key verification failed.
lost connection

Seuraavaksi päivitimme “server cache” -kohdan jotta voimme yhdistää kannettavan tietokoneen ja VPN -serverin:

 Sudo ssh-keygen -f “/root/.ssh/known_hosts” -R xxx.xx.xxx.x

Saimme yhdistettyä server -koneen kannettavan kanssa ja yhdistimme myös älypuhelimen kannettavalle. Siirrettiin uusi client tiedosto onnistuneesti kännykälle. Voimme nyt ottaa yhteyden VPN -serveriin mobiiliverkosta myös.      

### VPN:n testaaminen

Vaatimukset:

Käyttäjällä tulee olla openVPN -sovellus asennettuna ja oikea client.ovpn -tiedosto tallennettu laitteelle.

Käyttäjän kannettava on yhteydessä 4G -verkkoon, mutta se ei pysty löytämään VPN -serverin julkista- eikä yksityistä ip -osoitetta ping -komennolla. Ensiksi painetaan OpenVPN -ikonista, valitaan client profiili ja yhdistetään siihen. Kun yhteys toimii voidaan löytää OpenVPN -serverin ping komennolla. Voimme myös käyttää Putty -ohjelmaa ja ottaa yhteys OpenVPN -serveriin jopa labraverkon ulkopuolelta.

Mahdollisia tapoja kokeilla palvelua:

USB -tikku joka sisältää ohjetekstin, client.ovpn tiedoston ja OpenVPN -sovelluksen asennuskansion. 

Ohjeet sisältävät seuraavat kohdat:

-kuinka asentaa openVPN windows/linux/android -laitteille
-mihin client.ovpn tiedosto tulee siirtää
-mitä salaisella yhteydellä voi tehdä
-kuinka muodostaa yhteys


28.3.2018

## Luodaan “client.ovpn” tiedosto Hartolle


Otimme yhteyden VPN serveriin:

cd ~/openvpn-ca/
source vars
./build-key-pass harto

Asetimme salasanan tiedostolle ja enteriä muihin kysymyksiin, koska “vars” vastaa niihin.
Lopuksi “yes” kahteen varmistukseen.

cd ~/client-configs/
sudo ./make_config.sh harto
sudo ls ~/client-configs/files/

Nykyiset “client” -tiedostot:

client1.ovpn  client2.ovpn  harto.ovpn


11.4.2018

## LDAP testausta


Aloitimme asentamalla LDAP -server koneen joka sisältää käyttäjätiedot. OpenVPN -serverin tulee pystyä saamaan kirjautumistiedot LDAP -serveriltä kun tämä konfiguraatio on valmis. Ensiksi muutettiin osoitteita “hosts” -tiedostossa. Annettiin LDAP -client ja LDAP -server osoitteet.

nano /etc/hosts

Hosts tiedosto:

172.28.175.5 example.example server
172.28.175.1 labravpn client	

Seuraavaksi asennetaan LDAP:

sudo apt-get update
sudo apt-get -y install slapd ldap-utils

### Konfiguroidaan LDAP

Asennuksen jälkeen tekemämme konfiguraatiot:

	sudo dpkg-reconfigure slapd

No

Domain name = omavalintainen nimi

organisation name = -””-

LDAP admin salasana

Choose backend for LDAP = HDB

DB purge? = No

Move DB? = Yes

Konfiguraation lopussa saimme virheviestin, joten kokeilimme uudestaan. Tällä kertaa vastasimme kysymykseen “move old database” valinnalla “no”.

Seuraava kohta:

Allow LDAPv2? = no

Asennuksen jälkeen tulee verifioida LDAP:

sudo netstat -antup | grep -i 389

### LDAP domain konfiguraatio

Generoidaan “base.ldif” -tiedosto domainille:

$ nano base.ldif
	dn: ou=People,dc=ldaptest,dc=local
	objectClass: organizationalUnit
	ou: People

	dn: ou=Group,dc=ldaptest,dc=local
	objectClass: organizationalUnit
	ou: Group

Rakennetaan tiedostopolku:

ldapadd -x -W -D “cn=admin,dc=ldaptest,dc=local” -f base.ldif

Anna LDAP salasana.

### Lisätään LDAP käyttäjiä

Luodaan uusi LDIF käyttäjätiedosto uudelle käyttäjälle “ldapuser”:

nano ldapuser.ldif 

Liitä:
	
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

Seuraavaksi luodaan käyttäjä:

	ldapadd -x -W -D "cn=admin,dc=ldaptest,dc=local" -f ldapuser.ldif

Saatiin virheilmoitus “no such file or directory“. Ongelmana oli kirjoitusvirhe tiedoston nimessä. Korjasimme virheen ja kokeilimme uudestaan. Tämän jälkeen jatkettiin eteenpäin antamalla salasana:

	ldappasswd -s password123 -W -D "cn=admin,dc=ldaptest,dc=local" -x 
	"uid=ldapuser,ou=People,dc=ldaptest,dc=local"

-s käyttäjän salasana
-x minkä käyttäjänimen salasanaa on vaihdettu
-D nimi joka tullaan autentikoimaan LDAP -serverillä

Verifioidaan LDAP merkinnät

 	ldapsearch -x cn=ldapuser -b dc=itzgeek,dc=local

Enabloidaan LDAP login

Lähetetään LDAP tapahtumat log -tiedostoon /var/log/ldap.log
 
sudo nano /etc/rsyslog.d/50-default.conf
Lisää seuraava rivi tiedostoon:
local4.* /var/log/ldap.log
sudo service rsyslog restart
Serverin konfiguraation ollessa valmis aloitimme konfiguroimaan käyttäjäpuolta. Aloitimme asentamalla seuraavat kansiot:
 
sudo apt-get update
	sudo apt-get -y install libnss-ldap libpam-ldap ldap-utils nscd
 
Asennuksen jälkeen lisäsimme serverin ip -osoitteen ja porttinumeron kun niitä kysyttiin. Seuraavaksi me lisäsimme LDAP tietokannan domain -nimen (dc=ldaptest, dc=local). Seuraavaksi valitaan mikä LDAP versio tulee käyttöön, valitaan “3”. Seuraavana valitaan halutaanko LDAP -serveristä tehdä käyttäjän root database admin, mihin me vastattiin “no”. Lopuksi valitaan vielä halutaanko kirjautua palveluun kun halutaan hakea tietoa siitä, me valittiin “no” koska siihen ei ole vielä tarvetta. 
Näiden alkuasetusten jälkeen me muokkasimme seuraavaa tiedostoa:
sudo nano /etc/nsswitch.conf
Muokkausten jälkeen se näytti tältä:
#/etc/nsswitch.conf  
#  
#Example configuration of GNU Name Service Switch functionality.
#If you have the `glibc-doc-reference` and `info` packages installed, try:
#`info libc "Name Service Switch"` for information about this file.
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
Tämän jälkeen käynnistimme palvelun uudestaan:
sudo service nscd restart
Asennuksen jälkeen kokeilimme kirjautua server -koneeseen vasta tehdyllä käyttäjällä mutta se ei valitettavasti toiminut.


20.4.2018

## LDAP virtuaaliympäristössä


Ohje: 

https://blogs.msdn.microsoft.com/microsoftrservertigerteam/2017/04/10/step-by-step-guide-to-setup-ldaps-on-windows-server/ 

Luodaan virtuaalikone ja asennetaan siihen Windows Server 2016. Tämä tulee toimimaan LDAP -serverinä meidän OpenVPN -palvelun jatkona. Tässä serverissä tulee siis olemaan kirjautumistiedot joilla voidaan kirjautua OpenVPN -palveluun. OpenVPN -serveri tulee tarkistamaan nämä kirjautumistiedot ja päättää mitkä käyttäjät se voi päästää VPN -palveluun ja mitä oikeuksia heillä on. Tähän me käytimme Oracle VirtualBox -ohjelmaa. Tämän testin tarkoitus on kokeilla kuinka tämä toimii käytännön olosuhteissa. Me latasimme Microsoftin sivulta “evaluation” ISO -kuvakkeen joka on ilmainen. Asennuksen yhteydessä valittiin GUI jotta serverin käyttö olisi helpompaa.      

### Asennetaan LDAPS Windows Server 2016

Me seurasimme linkin ohjeita. Ensiksi tuli valita AD LDS -rooli. Tämän jälkeen me loimme AD LDS instanssin uudestaan Server Manager -ohjelmalla. Me seurasimme ohjeita paitsi kohdassa “importing LDIF files” jossa me valittiin kaikki vaihtoehdot.

Tämän jälkeen me avasimme ADSI Edit -attribuutin jonka avulla me yhdistettiin AD LDS instanssin CONTOSO :n. Seuraavaksi me avasimme yhteys -asetukset ja täytimme samat arvot kuin aikaisemminkin. Yhteys toimi ja me pystyimme nyt selaamaan hakemistoa “CN=MRS,DC=CONTOSO,DC=COM”.

Seuraavaksi luodaan Active Directory Certificate Services -palvelussa sertifikaatti LDAPS :lle. Tämän onnistumiseen meidät tuli luoda uusi rooli, johon me käytimme default asetuksia. Käynnistetään kone uudestaan. Tämä jälkeen avataan  “AD CS configuration” joka löytyy “AD CS page”. Luodaan sertifikaatti meidän serverille ohjeiden mukaan. Tässä vaiheessa tulee muistaa käyttää omia tietoja jne. Sertifikaatin ollessa valmis tuli verifioida että se oli olemassa “manage computer certificates” -kohdassa. 

Nyt tulee selvittää että host -koneen tilillä on pääsy privaattiin avaimeen. “Certutil utility” avulla voidaan löytää “Unique Container Name”. Komentokehotteen avulla ajetaan seuraava komento “certutil -verifystore MY” “Administrator” -käyttäjällä.  

Seuraavaksi mennään seuraavaan polkuun:

C:\ProgramData\Microsoft\Crypto\Keys\<UniqueContainerName>

ProgramData on piilotettu joten valitaan “view” -vaihtoehto ja siitä “Hidden items” jotta ProgramData tulee näkyviin. Kun tiedosto löytyy valitaan “properties” ja siitä “Security tab” ja sitten “edit“. Lisätään uusi ryhmä nimellä “NETWORK SERVICE” ja annetaan sille lukuoikeudet. 

Seuraavaksi siirretään sertifikaatti “CN=VPNPOJATLDAP” JER “key store” -kohtaan koska sitä ei ole allekirjoittanut luotettava “certificate authority”. Jotta siirto onnistuu käyttäen “keytool utility” meidän tulee ensin viedä sertifikaatti “.CER” -muotoisena “machine certificate store” -kohdasta. Valitse “Start”, etsi “Manage Computer Certificates” ja avaa se. Tämän jälkeen avataan “personal”, valitaan “VPNPOJATLDAP” -sertifikaatti ja valitaan “Export”. Tämän jälkeen avautuu “Certificate Export Wizard”. Seurattiin Wizardia loppuun asti perusasetuksilla ja toteutettiin export. Sertifikaatin pitäisi nyt olla siirretty. Seuraavaksi meidän tulisi siirtää se “JRE Keystore” -kohtaan käyttäen “keytool command” -kohtaa.

Varmistuakseemme että aikaisempi OpenLDAP -asennus meidän VPN -serverillä ei häiritsisi meidän uusia LDAP -server asetuksia me ajettiin seuraavat komennot VPN -serverillä:

            sudo apt-get purge ...
	sudo apt-get --purge remove ...

Poistettiin vanhat OpenLDAP tiedostot.


25.4.2018

## Serverin käyttäjien yhteys -lokien tarkastelua 


Komennolla sudo “cat /etc/openvpn/openvpn-status.log” pääsee tarkastelemaan muodostettuja VPN yhteyksiä. Loki näyttää millä “client.ovpn” tiedostolla yhteyksiä on muodostettu, sekä mistä ja milloin.

Huomasimme, että mikäli yksi laite on yhdistetty ensin käyttäen “client1” tiedostoa ja sen jälkeen toinen laite ottaa yhteyden käyttämällä samaa tiedostoa, näyttää loki ainoastaan jälkimmäisenä muodostetun yhteyden. VPN -yhteys ei kuitenkaan katkea ensimmäisen yhteyden muodostaneen laitteen ja serverin välillä. Se ei vain näy taulukossa. Eli tietoturvallisesti katsoen jokaiselle käyttäjälle tulisi luoda oma käyttäjätiedosto yhteyden muodostamista varten.

### OPENVPN serverin uudelleenluonti uudelle “Tuotantokoneelle” 

Tällä kertaa päivän tehtävissä oli asentaa aikaisemmin tekemämme toimiva OpenVPN -server toiselle koneelle jonne projekti tulee jäämään kurssin loppuessa talteen.
Meille oli valmiiksi luotu instanssi joka sisälsi Ubuntu 16.0.4 LTS serverin pohjana.
Aloimme aikaisemmin luomien ohjeiden perusteella asentamaan ja konfiguroimaan OpenVPN -palvelua normaalisti ja kaikki toimi ohjeiden mukaan. Ensimmäinen virhe oli tiedostoissa olevissa default asetuksissa. Meidän täytyi muuttaa ne osoittamaan luotuihin tiedostoihin. 

### Virhe käynnistäessä VPN palvelua

Yritimme käynnistää OpenVPN:n: 

sudo systemctl start openvpn@VPNSERVER

Saimme tämän virheilmoituksen: 

Seuraavalla komennolla saa lisätietoja virheestä.

journalctl -xe

Ongelma oli tiedostojen nimissä. Meillä oli jäänyt “VPNSERVER.conf” -tiedostoon default -tiedostonimiä, minkä takia OpenVPN ei löytänyt oikeita tiedostoja. 

Kävimme vaihtamassa tiedostojen “server” -nimet. Oikea nimi on “VPNSERVER”. Se valitti myös ettei löydä “ta.key” -tiedostoa. Selvisi että jostain syystä “ta.key” ei ollut kopioitunut aiemmin “/etc/openvpn” -hakemistoon. Kopioitua tiedoston sinne uudestaan saimme käynnistettyä OpenVPN :n.

Kun tarkistimme OpenVPN statuksen komennolla 

sudo systemctl status openvpn@VPNSERVER 

Saimme kuitenkin tiedon että palvelu ei ollut käynnistynyt oikein. Kyseessä oli kirjoitusvirhe. Annettua oikean komennon huomasimme että kaikki on toiminnassa. 


2.5.2018

## Windows Server 2016 asennus


Asensimme Windows Server 2016 koneeseen josta tulisi meidän LDAP -serveri. Käytimme USB tikkua jossa oli valmis ISO -kuva. Asennettiin desktop versio, jonka jälkeen aloitimme LDAP konfiguraation.

Konfiguraation kohdat:  

Manage -> Add roles and features
	
Ensimmäinen dia, paina next

Valitse Role-Based ja next

Valitse oma server and paina next
	
Merkkaa Active Directory Lightweight Directory Service ja next

Älä valitse mitään ja next

Next

Install ja close kun se on valmis

Nyt AD LDS rooli on luotu. Luodaan vielä uusi AD LDS instanssi “CONTOSO”. Painamalla “flag” -kuvaketta valitse “run the active directory lightweight directory service” avataksesi wizardin.

Next

Valitse “a unique instance”

Instanssin nimi: CONTOSO

Valitse default ports

Valitse “yes, create an application directory partition” ja lisää tämä partition nimeen: “CN=MRS,DC=CONTOSO,DC=COM” 

Käytä default määreitä “values” tallennuspaikoiksi

Valitse “network service account”

Valitse “yes” varoitusviestiin koska me käytämme vain yhtä LDAP serveriä. 
	
Valitse tämänhetkinen sisäänkirjautunut käyttäjä: admin
	
Merkkaa kaikki LDIF tiedostot

Varmista and jatka
	
Asennuksen ollessa valmis valitse close


