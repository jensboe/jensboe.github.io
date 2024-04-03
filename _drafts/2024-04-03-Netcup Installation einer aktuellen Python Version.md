---
layout: post
title:  "netcup: Installation einer aktuelleren Python-Version & Paketen"
date:   2024-04-04 10:00
categories: Website
tags: netcup Python
---

# Eine andere Python Version bei netcup nutzen

Zum Zeitpunkt des Schreibens (4.4.2024) bietet netcup die Python Version 3.7.3 und 2.7.16 an.
Diese sind am 25.3.2019 ([3.7.3](https://www.python.org/downloads/release/python-373/)) bzw. 4.3.2019 ([2.7.16](https://www.python.org/downloads/release/python-2716/)) erschienen.
Die Nachfolger schienen am 8.7.2019 ([3.7.4](https://www.python.org/downloads/release/python-374/)) bzw. 19.10.2019 ([2.7.17](https://www.python.org/downloads/release/python-2717/)).
Diese sind somit seit gut vier einhalb Jahren veraltet.

Eine Nachfrage beim Support ergab folgende Antwort:
>Wir stellen stets die aktuelle Python3-Version aus den Paketquellen des Betriebssystems bereit. Ein Upgrade von 3.7.3 auf 3.9.2 wird voraussichtlich in den nächsten Monaten erfolgen.

Nicht wirklich eine gute Nachricht.

Da ich bei netcup eine Webseite mit Django aufsetzen wollte, stellte sich aber gleich die nächste Frage.

# Bei netcup Python Pakete installieren

Wenn man sich via SSH mit dem Server verbindet, stellt man schnell fest, das `python` nicht funktioniert.

Sascha Szott stellt auf seiner [Website](https://saschaszott.github.io/2021/02/14/netcup-python-webhosting.html) eine Weg vor.
Dabei installiert er Pakete lokal auf dem PC und läd sie anschließend aus dem `site-packages`-Ordner in das Verzeichnis der Webapplikation hoch.
Das funktioniert jedoch nicht bei Paketen, die vorkompilierten Code enthalten, wie es z.B. bei Pillow der Fall ist.
Zusätzlich wird das Applikationsverzeichnis durch viele Abhängigkeiten schnell unübersichtlich.

Allerdings bin ich dann auf einen [Forenbeitrag von mhs](https://forum.netcup.de/netcup-anwendungen/wcp-webhosting-control-panel/p156843-wsgi-python-mit-phusion-passenger-auf-webhosting-8000/#post156843) gestoßen.
Mit seiner Schilderung habe ich es geschafft, eine aktuelle Python version bei netcup zu installieren und ich kann nun pip bei netcup nutzen.

#  Anleitung zum Installieren von Paketen bei netcup

1.  Herunterladen von miniconda in der Version miniconda3-4.5.11-Linux-x86_64.sh.

    Diese Version führte bei mhs und bei mir zum Erfolg.
    Neuere Versionen konnte ich in späteren Schritten nicht installieren.

2.  Hochladen via FTP

3.  Datei ausführen mit bash Minic...... dabei folgende Optionen....

4.  Datei .profile anlegen
```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```
5.  Ausloggen und wieder einloggen.

6.  Mit `conda update -n base -c defaults conda` kann nun Conda auf eine neue Version aktualisiert werden werden.

7.  Mit `conda create -n myWebsite python=3.12` kann nun z.B. eine virtuelle Umgebung für eine Webseite erstellt werden.
    In dieser läuft dann Python in der Version 3.12

8. mit `conda activate myWebsite` kann die virtuelle Umgebung aktiviert werden.

9. Pakete können nun mit `conda install <paket>` installiert werden.
    `pip` ist ebenfalls verfügbar.
    Allerdings berücksichtigt `pip install` die virtuelle Umgebung nicht und installiert die Pakete global.
    Dies kann man umgehen indem man `python -m pip` nutzt.

# Inhalt der pessanger_wsgi.py

Nachdem die Vorarbiet geleistet ist, muss nun die pessenger_wsgi angepasst werden.
Diese wird nämlich noch mit der ursprünglichen Python Version 3.7.3 aufgerufen.

hier msus dann der Inhalt meiner pessenger_wsgi hineinkopiert werden.




