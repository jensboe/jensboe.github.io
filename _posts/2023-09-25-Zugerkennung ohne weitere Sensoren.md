---
layout: post
title:  "Zugerkennung ohne weitere Sensoren"
date:   2023-09-25 8:00
categories: Elektronik Modellbahn
mermaid: true
tags: Modellbahn Theorie Software
---
Die meisten Zugerkennungen, die man im Internet für eine Analoge Modellbahn nutzen kann, nutzen Lichtschranken oder Gleiskontakte.
Auf diesen Aufwand kann verzichtet werden, Sofern man statt klassische Relais schnelle Solid State Relais (SSR) und einen Shunt oder ähnliches zur Strommessung verwendet. SSR haben üblicherweise Schaltzeiten im Bereich <10ms[^TLP241A], so dass eine Aktivierung oder Deaktivierung einzelner Blöcke in extrem kurzer Zeit durchgeführt werden kann.

Streng genommen werden dabei keine Züge erkannt, sondern Verbraucher innerhalb eines Blocks. Erst durch die Kombination der Blockzustände und den gegebenen Anfangszustand kann aus den Blockzuständen ermittelt werden, welcher Zug in welchem Block ist.

## Erkennung des Blockzustandes

Grundsätzlich kann der der Blockzustand gemäß dem nachfolgendem Ablaufdiagramm ermittelt werden. Die Deaktivierung und Aktivierung kann in einem Schritt durchgeführt werden, die Wartezeit für die Umschaltung sollte der maximalen Einschaltzeit der verwendeten SSR entsprechen. Zusätzlich muss nach dem Aktivieren der H-Bücke gewartet werden, da aufgrund der Induktivitäten nicht sofort mit einem ausreichendem Stromfluss gerechnet werden kann. Die genauen Wartezeiten hierfür müssen durch Messungen ermittelt werden.

Ein Problem stellen eventuelle kurzzeitige Unterbrechungen durch eine schlechte Verbindungen zwischen Zug und Gleis dar. So muss die Messung mehrfach durchgeführt werden, um einen Block sicher als "frei" zu klassifizieren, ein einmal gemessener Strom oberhalb der Detektionsschwelle klassifiziert einen Block hingegen umgehend als "belegt".

``` mermaid
graph TD

Start((Start))
disable_all_blocks[Deaktivierung aller Blöcke]
enable_test_block[Aktivierung des zu testenden Blocks]
wait_switch_time[Schaltzeit der SSR abwarten]
activate_DC[H-Brücke mit DC-Spannung aktivieren]
is_current_detected{Strom über
Schwellwert?}
blocked[Block belegt]
free[Block frei]
End((Ende))

Start --> disable_all_blocks --> enable_test_block --> wait_switch_time --> activate_DC --> is_current_detected

is_current_detected --> |Nein| free
is_current_detected --> |Ja| blocked
free --> End
blocked --> End
```

## Optimierung der Blockerkennung

Anstatt alle Blöcke reih um zu testen, macht es Sinn, nur die Blöcke zu testen, bei deinen eine Zustandsänderung möglich bzw. erwartet wird, weil Züge in ihn einfahren oder ihn verlassen oder zumindest die Möglichkeit oder Gefahr besteht. Dies kann der ganz normale Fahrbetrieb oder aufgrund eines Fehlers wie z.B. eine falsch stehende Weiche sein.

Für die freien Blöcke können dabei gruppiert zusammen getestet werden. Solange kein Strom oberhalb des Schwellwerts gemessen wird, sind alle Blöcke frei. Sobald ein Strom oberhalb des Schwellwerts gemessen wird erfolgt eine binäre Suche nach dem Block mit dem belegtem Gleis.

``` mermaid
graph TD

Start([Start])
mark_as_untested[Blöcke als
ungetestet markieren]
switch_blocks[Blockauswahl schalten
und bestromen]
is_current_detected{Strom über
Schwellwert?}
free[Blöcke frei
markieren]
more_than_one{Mehr als
einen Block
getestet?}

blocked[Block als
belegt markieren]
remaining_blocks{Ungeteste
Blöcke
vorhanden?}
reduce_blocks[Blockauswahl
halbieren]
End([Ende])

Start --> mark_as_untested --> switch_blocks --> is_current_detected
is_current_detected --> |Nein| free
is_current_detected --> |Ja| more_than_one
more_than_one --> |Nein| blocked
more_than_one --> |Ja| reduce_blocks
free --> remaining_blocks
blocked --> remaining_blocks
remaining_blocks --> |Nein| End
remaining_blocks -->|Ja| reduce_blocks
reduce_blocks --> switch_blocks

```

[^TLP241A]: TLP241A, Turn on time: 2,8ms (max. 5ms), Turn off time: 0,3ms(max. 1ms) [Datenblatt, S. 5](https://toshiba.semicon-storage.com/info/TLP241A_datasheet_en_20230525.pdf?did=14237&prodName=TLP241A)
