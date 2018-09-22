+++
title = "SMD-Reflow-Ofen vom Sperrmüll"
date = 2018-04-01T00:02:36+02:00
draft = false
tags = ["electronic"]
categories = ["PCB", "SMT"]
+++

![Mein SMD-reflow-Ofen](/images/reflow-800.jpg)

Umbau eines einfachen Mini-Backofens in einen Reflow-Ofen zum Löten von
SMD-Platinen.

<!--more-->

Einfacher Mini-Backofen mit vier Heizelementen, je zwei oben und unten,
zusammen 1200W, mechanischem Temperatur-Regler und mechanischer
Zeitschaltuhr.

Die Zeitschaltuhr und Temperaturregelung fliegen raus. Die Heizelemente
werden stattdessen von einem Solid-State-Relais geschaltet (mechanische
Relais hätten es auch getan). Die Glimmlampe im Power-Lämpchen wird durch
eine rote LED zur Anzeige des Relais-Status ersetzt.

An der Seite ist eine kleine Übertemperatur-Sicherung 216°C/10A/4mm an die
Ofenwand geschraubt. Für bleihaltiges Löten genügt diese Maximaltemperatur,
für bleifreies Löten ist ein Austausch vermutlich notwendig.

Alternativen bei [Reichelt](https://www.reichelt.de):
- MTS 240 Temperatursicherung, 10A, 240°C, 0,48€
- MTS 228 Temperatursicherung, 10A, 228°C, 0,48€

[Pollin](https://www.pollin.de):
- 260252	Temperatursicherung 227°C, 0,45€
- 260253	Temperatursicherung 240°C, 0,45€


## Temperatur-Profil

{{< figure src="https://i.stack.imgur.com/PhUW1.jpg"
caption="Kester-Pprofil für bleihaltige Lötpaste" >}}

Ein übliches Temperatur-Profil zum bleihaltigen Löten basiert auf diesen
Temperaturen und Zeiten:

1. ramp to soak (20°C->150°C): 1-3K/s (<1.8K/s ideal)
2. preheat/soak (150°C): const
3. ramp to peak (bis 210°C..225°C): 1-3K/s
4. cooling: so schnell wie möglich durch Öffnen der Klappe

- soaking zone: typ. 30..60s, max. 120s
- time to peak temp: typ. 210..300s, max. 330s
- reflow zone (Zeit mit T>180°C): typ. 45..75s


## Software

Regelungstechnisch ist der Ofen ein PT2-System. Theoretisch ist deshalb ein
PID-Regler die ideale Lösung, aufgrund der großen Verzögerungen hat in der
Praxis ein kombinierter Regel- und Steueralgorithmus aber besser
funktioniert.

Das Temperaturprofil ist in Phasen eingeteilt. Die Phasenübergänge sind
teilweise temperatur- und teilweile zeitabhängig. Eine Anpassung an den
jeweiligen Ofen ist deshalb notwendig.




## Hardware

Die Elektronik wird der Einfachheit halber aus fertigen Modulen
zusammengesetzt: Arduino Uno ($3,50), LCD-Keypad-Shield ($2,50) und ein
MAX6675-Breakout-Board ($2,20) zum Anschluss eines Typ-K Thermoelements
($1,00).

Das MAX6675-Board und die Anschlüsse für den Sensor und das Relais sind auf
einem eigenen Shield auf Lochraster aufgebaut und im Frontgehäuse montiert:

{{< figure src="/images/reflow-shield-schematic.png" caption="Reflow-Shield" >}}

Die Heizelemente werden mit einem Solid-State-Relais direkt im Ofengehäuse
geschaltet, einfache Relais-Module ($0,60) würden aber genausogut
funktionieren.  Dann kostet alles zusammen weniger als 10 Dollar bei
[Aliexpress](https://www.aliexpress.com).

Das LCD Keypad Shield verwendet diese Signale:

Arduino Pin	|Signal
--------	|--------
D4		|LCD D4
D5~		|LCD D5
D6~		|LCD D6
D7		|LCD D7
D8		|LCD RS
D9~		|LCD EN
D10~		|LCD Backlight
A0		|Spannungsteiler mit den Tasten

Mein Reflow-Shield enthält diese Verbindungen:

Arduino Pin	|Signal
--------	|--------
D13/SCK		|MAX SCK
D12/MISO	|MAX SO
D2		|MAX CS
A1		|Diode A
D3~		|HEAT_TOP (oder alles)
D16		|HEAT_BOTTOM (nicht benutzt)
D17		|HEAT_LED

Einige Pins sind noch frei:

Arduino Pin	|Signal
--------	|--------
D11~/MOSI	|frei
D18/A4/SDA	|frei
D19/A5/SCL	|frei

D12 und D13 könnten durch Doppelbelegung mit LCD noch freigeschaufelt
werden.

Initialisierung des LCD: `LiquidCrystal lcd(8, 9, 4, 5, 6, 7);`



## Stückliste und Preise

Temperatursicherung:

- Reichelt MTS 240 Temperatursicherung, 10A, 240°C, 0,48€
- Reichelt MTS 228 Temperatursicherung, 10A, 228°C, 0,48€
- Pollin 260252	Temperatursicherung 227°C, 0,45€
- Pollin 260253	Temperatursicherung 240°C, 0,45€

Steuerelektronik (alles von Aliexpress):

- Arduino Uno ($3,50)
- LCD-Keypad-Shield ($2,50)
- MAX6675-Breakout-Board ($2,20)
- Typ-K Thermoelements, 92cm (mit gelbem Stecker) ($1,00)
- Solid-State-Relais oder
- einfaches Relais-Modul für Arduino ($0,60)

Gehäuse:

- [Pollin 021-002-173](https://www.pollin.de/p/kunststoffgehaeuse-021-002-173-460006) von Pollin
  (2,40€)

Alles zusammen deutlich unter 15€.




## Andere Lösungen


### Thomas Pfeiffers Reflow-Regelung

[Thomas Pfeiffers Reflow-Regelung](http://thomaspfeifer.net/backofen_smd_reflow.htm)
war das ursprüngliche Vorbild. Seine Platine liess sich aber nur sehr
schlecht in das Ofengehäuse einbauen, deshalb bin auf die jetzige Lösung mit
exterem Controllergehäuse umgeschwenkt.

Seine Regelung arbeitet mit einer sehr einfachen Zweipunkt-Regelung. Die
thermischen Verzögerungen meines Ofens waren aber viel zu groß für diesen
Ansatz. In der Soaking-Phase haben sich Temperaturschwankungen von über 50K
ergeben - nicht anwendbar.

Sehr einfach aufgebaut:

- Si-Diode als Temperatursensor
- kein Display
- Solid-State-Relais Sharp S202S11
- max. 2 Kanäle, aber nur 1 verwendet.


### ControLeo2

[Peter Easton](https://github.com/engineertype) beschreibt in seiner
erstklassigen Anleitung sehr detailliert, wie ein perfekter Ofen-Umbau
aussehen sollte: [neue, sehr
perfektionistische](http://www.whizoo.com/reflowoven) und [alte, etwas
weniger anspruchsvolle](http://www.whizoo.com/c2) Version.

Trotz seiner Mahnungen habe ich auf die Beschichtung des Innenraums mit
Goldfolie und die Abdichtung kleiner Gehäuseöffnungen verzichtet, da ich nur
kleine Platinenformate bleihaltig löten möchte - da sind die Anforderungen
geringer.

Er hat leider nur die
[Software](https://github.com/engineertype/ControLeo2), nicht aber die
Hardware veröffentlicht. Die von mir zusammengestellte Hardware ist seiner
aber recht ähnlich geworden. Eine Portierung wäre nicht kompliziert, aber
erst sinnvoll, wenn ich Ober- und Unterhitze getrennt ansteuerbar machen
würde.

Als Arduino-Lib geliefert, die Anwendung ist in examples/.

Verwendet eine vereinfachte Version von LiquidCrystal: Feste Pinzuordnung,
nur 4 bit, kein rw-Pin.



#### ReflowOven

Ursprungsversion.

- einfacher Temperatur-basierter Ansatz ähnlich laminator
- hat schon die 8-Segment-Ansteuerung als mini-PWM.



#### ReflowOven2

Kompletter rewrite durch jemand anderen.
LCD16x2, 2 Taster. Interessanter Ansatz.

- phasenbasiert, ähnlich meiner Software.
- Die Phasen sind im EEPROM.
- Unterstützt 3 Heizelemente.
- Ansteuerung durch 8-Bit-Wert konfigurierbar, ergibt 9 Stufen 0-8.
- Sekundentakt für die einzelnen Bits.
- exakt abwechselndes schalten möglich um den Spitzenverbrauch zu
  kontrollieren.


#### ReflowWizard

Selbstlernend. Statt des starren 8-Bit-Musters für die Heizelemente kann der
Duty-Cycle in 100 Schritten eingestellt werden.

- 100 PWM-Stufen, 5s Periodendauer, 50ms/Stufe
- wenn eine Phase zu lange dauert, wird der Duty-Cycle erhöht. Ein wenig,
  stark oder sehr stark, je nach Abweichung.
- hat nach REFLOW (max. 90s) erst WAITING (40s), bevor er die Tür zum kühlen
  öffnet. Entspricht meinem COASTING.
- verwendet nicht die normale Servo-Lib, sondern etwas eigenes.
- spielt kurze Melodien, wenn er durch ist



## Temperatursensor

Thomas Pfeiffer verwendet eine gewöhnliche Silizium-Diode als
Temperatursensor. Er misst die Durchlassspannung der Diode an einem
ADC-Eingang des Arduino und leitet daraus die Temperatur ab. Das
funktioniert tatsächlich sehr gut und ist erstaunlich präzise, da dieser
Temperaturkoeffizient eine wohldefinierte physikalische Materialkonstante
des Siliziums ist.

ControLeo2 verwendet ein richtiges Thermoelement. Hauptvorteile sind:

- keine Kalibrierung notwendig, es ergeben sich direkt absolute Messwerte
- extrem schnelle Erfassung von Temperaturänderungen durch die winzige Masse
  des Thermoelements.

Ich habe zum Vergleich beide Sensoren eingebaut und die Resultate
verglichen. Die Skalierung der Diode passt nicht richtig, aber das
Verhalten ist trotzdem gut zu erkennen:

{{< figure src="/images/reflow-600.png" caption="Reflow soldering a PCB." >}}

Bei statischen Messungen liefern beide vergleichbar gute Ergebnisse, bei
schnellen Temperaturänderungen hinkt die Diode aber immer deutlich
hinterher. Da es hier um genau diese schnellen Änderungen gehen soll, habe
ich am Ende dann doch wie beim ControLeo2 ein richtiges Thermoelement
verwendet. 


## Hintergrundinfos

Präzise Temperaturmessung mit der pn-Strecke eines Transistors weitgehend
unabhängig von Fertigungstoleranzen durch eine delta-Messung:
[Linear Technology AN137](http://www.analog.com/media/en/technical-documentation/application-notes/an137f.pdf)

