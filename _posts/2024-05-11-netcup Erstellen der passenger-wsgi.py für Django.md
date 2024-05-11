---
layout: post
title:  "netcup: Erstellen der passenger-wsgi.py für Django"
date:   2024-04-05 20:00
categories: Website
tags: netcup Python

---

Zum [vollwertiges Python und pip bei netcup nutzen]({%post_url 2024-04-04-Netcup Installation einer aktuellen Python Version %}) habe ich einen eigenen Beitrag geschrieben, hier wird nun der Inhalt der passenger-wsgi.py beschrieben.

# Voraussetzungen

Für jede Webseite lege ich eine seperate Conda Umgebung an.
Für diese Beispiel verwendet ich `mywebsite`.


# Korrekten Interpreter aufrufen.
Der User, der die pessanger-wsgi.py ausführt ist nicht der FTP User.
Daher lässt sich der Python Interpreter nicht so einfach finden, zusätzlich müssen wir ihn direkt aufrufen.
Dazu verwende ich nachfolgendes Script, welches mir für spätere Updates oder weitere Seiten eine einfach Anpassung ermöglicht.
```python
import os
import sys

APP_SPECIFIC_VENV = "jfm_test"
PYTHON_VERSION = "python3.12"
MINICONDA_ROOT = "/miniconda3"

INTERP = os.environ["HOME"]+MINICONDA_ROOT+"/envs/"+APP_SPECIFIC_VENV+"/bin/"+PYTHON_VERSION
```
Via `sys.executable` kann der Pfad des aktuellen Interpreters herausgefunden werden.
Beim ersten Aufruf wird der Interpreter nicht dem Pfad entsprechen.
In dem Fall Rufen wir mit `os.execl` den korrekten Interpreter auf.
```python
if sys.executable != INTERP:
    os.execl(INTERP, INTERP, *sys.argv)
```
Nun kann der reguläre Code einer wsgi Datei von Django folgen.
DAbei muss der Pfad zum `DJANGO_SETTINGS_MODULE` natürlich noch angepasst werden.
```python
from django.core.wsgi import get_wsgi_application # pylint: disable=wrong-import-position

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'path.site.settings')
application = get_wsgi_application()
```

# Python im Webhosting Control Panel aktivieren

Python muss für die entsprechende Seite im Webhosting Control Panel eingeschaltet werden.
Der Modus sollte nur zur Testphase auf `Entwicklung` stehen, da im Fehlerfall viele Informationen angezeigt werden, die besser nicht in einem Produktivsystem angezeigt werden sollten.

![Python Einstellungen im Webhosting Controll panel](/assets/posts/netcup_python/python_settings.png)

Ruft man die entsprechende Seite nun im Browser auf, erwartet einen höchst wahrscheinlich die Fehlerseite von Passenger.
Dies liegt daran, das django und andere Pakete noch nicht installiert wurden.
Dies kann behoben werden, indem man sich via SSH einloggt, die Conda Umgebung via `conda activate myWebsite` aktiviert und anschließend die fehlenden Pakete via `python -m pip install <package>` nachinstalliert.
Alternativ installiert man die Python automatisch, das beschreibe ich in einem nachfolgendem Artikel.

![Fehlerseite von Passenger](/assets/posts/netcup_python/passenger_error_screen.png)
