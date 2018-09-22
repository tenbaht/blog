+++
title = "SMD reflow oven: lessons learned"
date = 2018-04-02T12:05:00+02:00
draft = false
tags = ["electronic", "reflow oven"]
categories = ["PCB", "SMT"]
+++


(Sorry, no english version yet.)

Der Ofen funktioniert und tut genau das, was er soll. Gut. Aber der Weg
dorthin war länger und umständlicher als er hätte sein müssen.

## Was ich heute anders machen würde


### Immer eine eigene Platine

Am Ende sehe ich mich wieder bestätigt: Niemals ohne richtige Platine.
Fertige Komponenten sind extrem nützlich für schnelle Versuchsaufbauten,
aber für ein fertiges Gerät ergeben sich zu viele Kompromisse und am Ende
dauert der Aufbau sogar länger, als wenn ich gleich eine spezielle Platine
gemacht hätte.

Probleme hier:

- Display steht auf dem Kopf. Entweder müsste ich das USB-Kabel nach links
  aus dem Gehäuse führen (was dann die Tür blockiert) oder das Display steht
  Kopf - beides ziemlich bekloppt.
- Unsaubere Optik der Tastenkappen. Die Taster auf dem Keypad-Shield haben
  keinen ausreichend langen Tastarm, um eine Tastenkappe montieren zu
  können. Ich musste mich mit zweckentfremdeten M3-Schrauben und Heisskleber
  behelfen. Da ist nichts gerade.
- Schlechte Auflösung der analogen Temperatur-Messung mit Diode.
  Das keypad-Shield benötigt einen ADC-Kanal mit 5V Referenzspannung zur
  Tastenabfrage. Für die Dioden-Lösung wäre aber eine Referenz von 1.1V
  wesentlich sinnvoller. So reduziert sich die Genauigkeit der
  Spannungsmessung für die Temperatur-Mess-Diode fast um Faktor 5.
  (Nur 5mV/Schritt statt 1.1mV/Schritt). Die mögliche Temperaturauflösung
  reduziert sich auf ca. 2.5K statt der eigentlich möglichen 0.5K.
- lieber hätte ich das Gehäuse senkrecht montiert und ein kleines
  graphisches Display eingebaut. Dafür gibt es aber kein Shield. Wenn ich
  das aufbaue, kann ich aber auch gleich alles auf eine Platine setzen.

Um das Problem mit der ADC-Referenz-Spannung könnte ich vielleicht noch
drumrumbasteln.  Die Diode muss ja nur maximal einmal pro Sekunde abgefragt
werden, da könnte ich auch die ADC-Referenzspannung umschalten und die
notwendige Wartezeit in Kauf nehmen.  Oder die Zeit, die ich zum Austüfteln
von solchem Murks brauche lieber gleich in eine richtige Platine
investieren.


#### Anderes Display

Besser wäre es, das Gehäuse senkrecht zu montieren. Das erfordert ein
weniger breites Display - ein kleines graphisches Display würde sich
anbieten.


#### Andere CPU

Graphisches Display bedeutet größerer Speicherbedarf - warum also nicht
gleich ein kleines STM32-Modul (Bluepill/Redpill) einsetzen? Kostet sogar
weniger als ein Arduino Nano und ist auch nicht komplizierter zu verwenden.
Eigentlich gibt es heute wirklich kaum noch Gründe, überhaupt noch auf AVRs
zu setzen - außer, dass sie so schön übersichtlich sind.


#### Relais statt Opto-Triac

Der Nulldurchgangsdetektor im verwendeten Opto-Triac S202S02 erlaubt eine sehr
elegante, feinfühlige und trotzdem störungsarme Modulation der Heizleistung.

Der schließlich verwendete Regel-Algorithmus nutzt diese Möglichkeit aber
gar nicht und schaltet doch nur vier Mal pro Lötzyklus - das hätte ein
einfaches Relais-Modul aus China ganz ohne Lötarbeiten auch geschafft (hier
also doch wieder ein Fertigmodul!)


#### Wenn Triac, dann etwas lieferbares

Das Opto-Triac S202S02 wird nicht mehr produziert. Es ist mittlerweile schwer
erhältlich und unverhältnismäßig teuer (selbst bei aliexpress).

Wenn schon löten, dann besser einen einfaches Opto-Triac ohne Leistungsteil
wie z.B. etwas aus der MOC3xxx-Baureihe mit einem normalen Triac verwenden. 
Viel günstiger und jederzeit problemlos erhältlich.
