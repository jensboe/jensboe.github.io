---
layout: post
title:  "netcup: Erstellen der passenger-wsgi.py für Django"
date:   2024-04-05 20:00
categories: Website
tags: netcup Python pip miniconda3 Django

---

Nachdem ich bereits einen Beitrag zum Thema [vollwertiges Python und pip bei netcup nutzen]({% post_url 2024-04-04-Netcup Installation einer aktuellen Python Version %}) verfasst habe, möchte ich nun den Inhalt der passenger-wsgi.py näher erläutern.

## Voraussetzungen

Für jede meiner Webseiten erstelle ich eine separate Conda-Umgebung.
In diesem Beispiel verwende ich `mywebsite`.

## Korrekten Interpreter aufrufen

Der Benutzer, der die passenger-wsgi.py ausführt, ist nicht der FTP-Benutzer.
Daher ist der Python-Interpreter nicht ohne Weiteres auffindbar, und wir müssen ihn direkt mit dem vollen Pfad aufrufen.
Das folgende Skript erleichtert mir spätere Updates oder Anpassungen für weitere Seiten:

```python
import os
import sys

APP_SPECIFIC_VENV = "jfm_test"
PYTHON_VERSION = "python3.12"
MINICONDA_ROOT = "/miniconda3"

INTERP = os.environ["HOME"]+MINICONDA_ROOT+"/envs/"+APP_SPECIFIC_VENV+"/bin/"+PYTHON_VERSION
```

Mithilfe von `sys.executable` lässt sich der Pfad des aktuell genutzten Interpreters ermitteln.
Stimmt dieser beim ersten Aufruf nicht mit dem gewünschten Pfad überein, rufen wir mit `os.execl` den korrekten Interpreter auf:

```python
if sys.executable != INTERP:
    os.execl(INTERP, INTERP, *sys.argv)
```

Anschließend folgt der reguläre Code einer Django-wsgi-Datei.
Der Pfad zum `DJANGO_SETTINGS_MODULE` muss entsprechend angepasst werden:

```python
from django.core.wsgi import get_wsgi_application # pylint: disable=wrong-import-position

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mywebsite.settings')
application = get_wsgi_application()
```

## Python im Webhosting Control Panel aktivieren

Python muss im Webhosting Control Panel von netcup für die betreffende Seite aktiviert werden.
Der Modus sollte während der Testphase auf `Entwicklung` gesetzt sein, da im Fehlerfall viele Informationen angezeigt werden, die besser nicht in einem Produktivsystem angezeigt werden sollten.

![Python Einstellungen im Webhosting Controll panel](/assets/posts/netcup_python/python_settings.png)

Ruft man die entsprechende Seite nun im Browser auf, erwartet einen höchst wahrscheinlich die Fehlerseite von Passenger.
Dies liegt daran, das django und andere Pakete noch nicht installiert wurden.
Dies kann behoben werden, indem man sich via SSH einloggt, die Conda Umgebung via `conda activate myWebsite` aktiviert und anschließend die fehlenden Pakete via `python -m pip install <package>` nachinstalliert.
Alternativ installiert man die Python automatisch, das beschreibe ich in einem nachfolgendem Artikel.

![Fehlerseite von Passenger](/assets/posts/netcup_python/passenger_error_screen.png)