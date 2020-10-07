+++
title = "Akkupack vom Staubsaugerroboter wiederbleben"
date = 2019-11-14T21:52:29+01:00
draft = true
tags = []
categories = []
+++

Unser Staubsaugerroboter Vileda A3 hat nach einigen Jahren eifrigen
Staubsaugens plötzlich damit begonnen, sich schon nach sehr kurzer Zeit
(max. 1 Minute) mit blinkender Akku-LED auszuschalten. Die Spannung am Akku
sieht gut aus: Gut 16,5V im Leerlauf, nur minimal weniger unter (leichter)
Belastung. Trotzdem ist der Roboter offenbar der Meinung, der Akku sei leer.

Der Staubsauger verwendet ein 14,4V Akkupack aus 12 NiMH-Zellen mit 1700mAh.
Kompatible Ersatzakkus gibt es bei Amazon schon ab 24€, trotzdem möchte ich
erst eine Wiederbelebung versuchen.

![Akkupack Vom Staubsaugerroboter Wiederbleben](/images/titelbild.jpg)
{{< figure src="/images/titelbild.jpg" caption="Akkupack Vom Staubsaugerroboter Wiederbleben" >}}

<!--more-->

Bernhard Abmayr hat auf [seiner
Seite](http://www.edv-abmayr.de/edv/service/nimh_akku/nimh_akku_reparatur.htm)
beschrieben, wie er durch mehrmaliges kontrolliertes und sehr langsames
Entladen des Akkus dieses Problem in den Griff bekommen hat.

Sein Vorschlag: Den Akku ganz langsam mit 1/100 C entladen bis zu einer
Entladeschlussspannung von 0,7 bis 1,0V pro Zelle. Entscheidend ist es dabei
die Spannungen aller Zellen einzeln zu prüfen und nicht nur die
Gesamtspannung des Akkupacks um so einzelne faule Zellen identifizieren zu
können.

Die Wikipedia erwähnt den 
[Batterieträgheitseffekt](https://de.wikipedia.org/wiki/Nickel-Metallhydrid-Akkumulator#Batterietr%C3%A4gheitseffekt)
von NiMH-Akkus. Er bewirkt eine Absenkung der Akkuspannung und könnte das
Fehlerbild auch erklären. Auch dieser Effekt soll durch mehrmalige
Lade/Entladezyklen beseitigt werden können. Erst Entladung mit 1/10 C bis
ca. 0,9V, danach noch vier mal Laden und mit 0,5 bis 1 C entladen.


Voll 1,4V, mittig 1,2V, Entladeschluss 1,0V oder 0,9V.
