+++
title = "freetz auf einer Fritzbox 3170 installieren"
date = 2018-10-02T22:16:28+02:00
draft = false
categories = ["network"]
+++


Meine aktuelle Fritzbox macht Probleme, deshalb will ich die uralte 3170
zumindest vorübergehend wieder in Betrieb nehmen. Freetz bietet mehr
Möglichkeiten als die normale Firmware, also mal ausprobieren.


## freetz-Image compilieren

Es gibt keine fertigen Images, sondern es muss erst selbst aus den
svn-Sourcen compiliert werden.  Das build-System von stable-2.0 ist total
veraltet und lässt sich nur mit gcc bis zur Version 5.x compilieren. 
Besser trunk verwenden, der funktioniert auch mit gcc-7.0.

Mein Basissystem ist Linux Mint 19 (Ubuntu 18.04), 64 Bit. Nötige Pakete
installieren:

	# nötige Pakete:
	apt install subversion libtool-bin
	# für trunk zusätzlich noch:
	apt install acl-dev libcap-dev libreadline-dev
	# auf 64-Bit System zusätzlich noch:
	apt install libncurses-dev gcc-multilib libc6-dev-i386

System runterladen und konfigurieren:

	svn checkout http://svn.freetz.org/trunk
	cd trunk
	make menuconfig

Unter "Webinterfaces->AVM Firewall" aktivieren.  Speicher ist leider extrem
knapp und es müssen als Ausgleich ein paar andere Funktionen deaktiviert
werden.  Ich habe Samba, FTP und Musikserver abgeschaltet, da ich eh' keinen
USB-Stick mit der Box verwenden werde.

System compilieren (dauert ein paar Minuten):

	make

Fertiges Image findet sich in `images/`, die zum Bauen verwendete
Konfiguration in `.config`. Das Makefile lädt das Original-Firmware-Image
von der AVM-Seite nach `dl/fw/fritz.box_wlan_3170.49.04.58.image`



## Flashen

Ganz normal als Firmware-Update.



## Einrichten

Telnet-Server aktivieren, einloggen (root/freetz), dann Sicherheitsstufe auf
0 (unbeschränkt) setzen:

	echo 0 > /tmp/flash/mod/security
	modsave
