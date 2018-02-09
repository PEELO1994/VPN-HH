# Muistiinpanoja



## VPN opiskelun tuloksia
 
### Älä käytä PPTP (point-to-point-protocol)!
PPTP on yleinen laajasti käytetty salaus protokolla, jota on käytetty jo Windows 95 saakka. PPTP on vanha ja täynnä haavoittuvuuksia. Niin sanottua salattua liikennettä pystyttäisiin helposti purkamaan, jolloin koko projektin idea (salattu yhteys) menee hukkaan. Helppo asentaa, mutta ei ole sen arvoista, ei käytetä.
 
### OpenVPN
OpenVPN on uudempi ratkaisu ja ei toistaiseksi ole sisältänyt suuria haavoittuvuuksia. Se on vapaan lisenssin ohjelmisto, joka käyttää TLS-protokollaa avainten vaihtamiseen jonka jälkeen salaus tapahtuu ESP-protokollan avulla. Tämä vaikuttaa eri artikkeleiden perusteella turvalliselta ja järkevältä VPN ratkaisulta. Yksi huono puoli on ainakin kolmannen osapuolen ohjelmisto mikä vaaditaan. OpenVPN:n pitäisi olla erittäin configurable, eli pääsisimme oikeasti muokkaamaan VPN:n haluamaksemme. 
 
### L2TP/IPsec
L2TP protokolla on VPN protokolla joka ei tarjoa itsessään salausta, vaan salaus suoritetaan usein IPsec:in avulla. Tämän ratkaisun huonona puolena on hitaus. L2TP tarkoittaa osi-mallin toisessa kerroksessa tapahtuvaa tunnelointia, mikä tarkoittaa sitä, että liikenne täytyy ensin muuttaa 2. kerroksen muotoon, jonka jälkeen se voidaan vasta salata. Ratkaisu ei myöskään pelaa hyvin yhteen tulimuurien kanssa, koska se käyttää UDP porttia 500, jonka takia sitä ei voida naamioida muiksi porteiksi. Hyvä puoli ratkaisussa on, että se on kuulemma helppo toteuttaa. Useat käyttöjärjestelmät ja älypuhelimet tänä päivänä sisältävät nämä ominaisuudet, jolloin tekniikan käyttöönotto ei vaadi suuria ponnistuksia. L2TP/IPsec pitäisi olla suhteellisen hyvä salaukseltaan, mutta on hitaampi kuin muut ratkaisut.

### L2Sec 

Tehtiin korjaamaan IPsecin turvallisuus viat. Suojaus mekanismit ovat vahvat ja perustuvat SSL/TLS käyttöön.

### SSTP 
Secure Socet Tunneling Protocol on Windowsille kehitetty tunnelointi protokolla, mikä käyttää SSLv3 salausta, niin kuin OpenVPN. Se mahdollistaa sujuvan liikenteen myös palomuurien läpi. SSTP soveltuu parhaiten Windowseille, mutta toimii muillakin alustoilla. Voidaan konfiguroida käyttäään AES (advanced encryption protocol) salausta, mikä on todella luotettavaa. 

### Loppupäätelmä
Uskoisin, että parhaat ratkaisut joita voisimme pyrkiä hyödyntämään on SSTP ja OpenVPN. OpenVPN:ssä huolestuttaa eniten kolmannen osapuolen ohjelmisto. SSTP ratkaisussa olisi järkevää käyttää Windowsia, mikä ei olisi yhtään huono idea. Asentaa palvelimelle esim. Windows Serverin (mikäli saadaan lisenssit(kokeiluaika lienee turha, kun halutaan pysyvä ratkaisu)). Nämä molemmat olivat kuitenkin hyvien salaus- ja palomuurilta piiloutumisominaisuuksiensa, johdosta mielestäni pahraita ehdokkaita näistä vaihtoehdoista. Joka tapauksessa asiaan pitää paneutua vielä syvemmin.

Luettavia Linkkejä:

https://en.wikipedia.org/wiki/Virtual_private_network
https://openvpn.net/index.php/open-source/documentation/howto.html





Lähteet: Wikipedia, https://www.howtogeek.com/211329/which-is-the-best-vpn-protocol-pptp-vs.-openvpn-vs.-l2tpipsec-vs.-sstp/


----------------------------------------------------------------------------------------------------------------------------------------
7.2.2018

# Opiskelua
Luimme kirjaa Beginning OpenVPN, josta lähdimme opiskelemaan yleistä käsitystä OpenVPN:n toiminnasta ja saisimme lisää käsitystä, siitä kuinka projektia lähdetään toteuttamaan.

Käytimme aikaa lukemalla kirjaa Beginning OpenVPN 2.0.9 : Build and Integrate Virtual Private Networks using OpenVPN, joka antoi hyvän perus käsityksen ylinpäätään VPN toimintatavoista. Kirja käsitteli millaisia protokollia OpenVPN käyttää toiminnassaan ja kuinka avain salaus ja vaihtaminen tapahtuu. Kirjasta löytyi myös ohjeita OpenVPN:n asentamiseen ja käyttöönottoon. 

Luimme myös Mikko Turpeisen opinnäytetyöytä, josta saimme lisätietoa OpenVPN:stä. Selvitimme millä eri tavoilla OpenVPN voi käyttää.

http://www.theseus.fi/handle/10024/73738


----------------------------------------------------------------------------------------------------------------------------------------
9.2.2018

Ohjeita OpenVPN:n käyttöön ottoon. Tältä sivulta löytyy todella hyviä ohjeita VPN käyttöönottoon liittyen. Käytännössä meidän täytyy asentaa labraverkkoon OpenVPN server ja käyttäjät jotka ottavat yhteyden luokkaan asentavat laitteilleen OpenVPN clientin. OpenVPN:ssä on mahdollista vaatia käyttäjätunnusta ja salasanaa, kun VPN yhteyttä muodostetaan, tulemme varmasti käyttämään sellaista ratkaisua. Ideaalista olisi jos pystyisimme hyödyntämään opiskelijoiden HH tunnuksia.
https://openvpn.net/index.php/open-source/documentation/howto.html 
