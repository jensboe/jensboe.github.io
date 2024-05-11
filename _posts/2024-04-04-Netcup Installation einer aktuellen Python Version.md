---
layout: post
title:  "netcup: Installation von Python Paketen und aktualisieren der Python Version"
date:   2024-04-04 18:00
categories: Website 

tags: netcup Python pip miniconda3 Django
---

Python ist bei vielen Webhostern ohne VPS nicht verfügbar, aber netcup bildet eine Ausnahme.
Dennoch gestaltet sich die Einrichtung einer vollständigen Python-Umgebung bei netcup nicht ganz einfach.
In diesem Leitfaden beschreibe ich, wie ich eine solche Umgebung bei netcup eingerichtet habe, insbesondere im Rahmen des Webhosting 4000-Angebots.

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

Wenn man sich via SSH mit dem Server verbindet, stellt man schnell fest, das der Befehl `python` nur eine Fehlermeldung ausgibt.

Sascha Szott stellt auf seiner [Website](https://saschaszott.github.io/2021/02/14/netcup-python-webhosting.html) eine Weg vor.
Dabei installiert er Pakete lokal auf dem PC und läd sie anschließend aus dem `site-packages`-Ordner in das Verzeichnis der Webapplikation hoch.
Das funktioniert jedoch nicht bei Paketen, die vorkompilierten Code enthalten, wie es z.B. bei Pillow der Fall ist.
Zusätzlich wird das Applikationsverzeichnis durch viele Abhängigkeiten schnell unübersichtlich.

Allerdings bin ich dann auf einen [Forenbeitrag von mhs](https://forum.netcup.de/netcup-anwendungen/wcp-webhosting-control-panel/p156843-wsgi-python-mit-phusion-passenger-auf-webhosting-8000/#post156843) gestoßen.
Mit seiner Schilderung habe ich es geschafft, eine aktuelle Python version bei netcup zu installieren und ich kann nun pip bei netcup nutzen.

#  Anleitung zum Installieren von Paketen bei netcup

1.  Ich habe Miniconda3 auf [repo.anaconda.com/miniconda/](https://repo.anaconda.com/miniconda/) heruntergeladen. Genauer die Version [miniconda3-4.5.11-Linux-x86_64.sh](https://repo.anaconda.com/miniconda/Miniconda3-4.5.11-Linux-x86_64.sh).
    Neuere Versionen haben bei mir in den späteren Schritten nicht funktioniert. Daher bin ich den Rat von mhs gefolgt und habe explizit die Version 4.6.11 heruntergeladen. In dne späteren Schritten wird miniconda3 auch noch auf eine neuere Version aktualisiert werden.

2.  Anschließend habe ich die Datei via FTP in das root Verzeichnis meines Webspace geladen.

3.  Nun habe ich mich via SSH mit dem Server verbunden und mit `bash Miniconda3-4.5.11-Linux-x86_64.sh` die Installation gestartet (falls eine andere Version installiert wird, muss der Dateiname selbstverständlich angepasst werden).
    Man kann nun die Lizenzvereinbarung lesen (weiter mit Enter) und diese am Ende mit `yes` oder `no` bestätigen bzw. ablehnen.
    Im Anschluss kann der Installationspfad angepasst werden, ich bin bei der Standardeinstellung `miniconda3` geblieben.
    Nach der Installation wird gefragt, ob man den Installationspfad zu `.bashrc` hinzufügen will.
    Dies bestätigen wir mit `yes`.
    Die Installation ist nun abgeschlossen.

4.  Nun  kann ich mich Anleitung aus SSH ausloggen.
    Allerdings würde bei einer neuen Verbindung noch kein Conda aktiv werden.
    Dazu muss ich erst die Datei `.profile` mit nachfolgendem Inhalt anzulegen.
```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```
    Das mache ich wieder per FTP und direkt im root Verzeichnis.

5.  Nun logge ich mich wieder ein.
    conda sollte nun Teil des Pfads sein. Dies kann ich mit `conda info` überprüfen.

    ```bash
    bash-5.0$ conda info

        active environment : None
                shell level : 0
        user config file : /.condarc
    populated config files :
            conda version : 4.5.11
        conda-build version : not installed
            python version : 3.7.0.final.0
        base environment : /miniconda3  (writable)
            channel URLs : https://repo.anaconda.com/pkgs/main/linux-64
                            https://repo.anaconda.com/pkgs/main/noarch
                            https://repo.anaconda.com/pkgs/free/linux-64
                            https://repo.anaconda.com/pkgs/free/noarch
                            https://repo.anaconda.com/pkgs/r/linux-64
                            https://repo.anaconda.com/pkgs/r/noarch
                            https://repo.anaconda.com/pkgs/pro/linux-64
                            https://repo.anaconda.com/pkgs/pro/noarch
            package cache : /miniconda3/pkgs
                            /.conda/pkgs
        envs directories : /miniconda3/envs
                            /.conda/envs
                platform : linux-64
                user-agent : conda/4.5.11 requests/2.19.1 CPython/3.7.0 Linux/4.19.0-21-amd64 / glibc/2.28
                    UID:GID : 31358:1003
                netrc file : None
            offline mode : False
    ```

6.  Mit `conda update -n base -c defaults conda` kann nun Conda aktualisiert werden werden.
    Dies dauert etwas.
    Zwischendurch muss ich mit `y` bestätigt werden, das die Pakete geupdatet werden sollen.

7.  Will man im Anschluss mit `conda info` prüfen, welche Version nun installiert wurde.
    Dabei kam es bei mir zu einem Fehler:

    ```bash
    bash-5.0$ conda info

    # >>>>>>>>>>>>>>>>>>>>>> ERROR REPORT <<<<<<<<<<<<<<<<<<<<<<

        Traceback (most recent call last):
        File "/miniconda3/lib/python3.10/site-packages/conda/exception_handler.py", line 17, in __call__
            return func(*args, **kwargs)
        File "/miniconda3/lib/python3.10/site-packages/conda/cli/main.py", line 78, in main_subshell
            exit_code = do_call(args, parser)
        File "/miniconda3/lib/python3.10/site-packages/conda/cli/conda_argparse.py", line 166, in do_call
            result = getattr(module, func_name)(args, parser)
        File "/miniconda3/lib/python3.10/site-packages/conda/cli/main_info.py", line 347, in execute
            info_dict = get_info_dict(args.system)
        File "/miniconda3/lib/python3.10/site-packages/conda/cli/main_info.py", line 147, in get_info_dict
            _supplement_index_with_system(virtual_pkg_index)
        File "/miniconda3/lib/python3.10/site-packages/conda/core/index.py", line 177, in _supplement_index_with_system
            for package in context.plugin_manager.get_virtual_packages():
        File "/miniconda3/lib/python3.10/site-packages/conda/plugins/manager.py", line 304, in get_virtual_packages
            return tuple(self.get_hook_results("virtual_packages"))
        File "/miniconda3/lib/python3.10/site-packages/conda/plugins/manager.py", line 181, in get_hook_results
            plugins = [item for items in hook() for item in items]
        File "/miniconda3/lib/python3.10/site-packages/conda/plugins/manager.py", line 181, in <listcomp>
            plugins = [item for items in hook() for item in items]
        File "/miniconda3/lib/python3.10/site-packages/conda/plugins/virtual_packages/cuda.py", line 65, in conda_virtual_packages
            cuda_version = cached_cuda_version()
        File "/miniconda3/lib/python3.10/site-packages/conda/plugins/virtual_packages/cuda.py", line 60, in cached_cuda_version
            return cuda_version()
        File "/miniconda3/lib/python3.10/site-packages/conda/plugins/virtual_packages/cuda.py", line 35, in cuda_version
            queue = context.SimpleQueue()
        File "/miniconda3/lib/python3.10/multiprocessing/context.py", line 113, in SimpleQueue
            return SimpleQueue(ctx=self.get_context())
        File "/miniconda3/lib/python3.10/multiprocessing/queues.py", line 342, in __init__
            self._rlock = ctx.Lock()
        File "/miniconda3/lib/python3.10/multiprocessing/context.py", line 68, in Lock
            return Lock(ctx=self.get_context())
        File "/miniconda3/lib/python3.10/multiprocessing/synchronize.py", line 162, in __init__
            SemLock.__init__(self, SEMAPHORE, 1, 1, ctx=ctx)
        File "/miniconda3/lib/python3.10/multiprocessing/synchronize.py", line 57, in __init__
            sl = self._semlock = _multiprocessing.SemLock(
        OSError: [Errno 38] Function not implemented

    `$ /miniconda3/bin/conda info`


    An unexpected error has occurred. Conda has prepared the above report.
    If you suspect this error is being caused by a malfunctioning plugin,
    consider using the --no-plugins option to turn off plugins.

    Example: conda --no-plugins install <package>

    Alternatively, you can set the CONDA_NO_PLUGINS environment variable on
    the command line to run the command without plugins enabled.

    Example: CONDA_NO_PLUGINS=true conda install <package>

    If submitted, this report will be used by core maintainers to improve
    future releases of conda.
    Would you like conda to send this report to the core maintainers? [y/N]:

    ```

    Der Hinweis bezüglich dem Setzen von Umgebungsvariablen brachte mir keine Verbesserung.


    Stattdessen habe ich die Datei `/miniconda3/lib/python3.10/site-packages/conda/plugins/virtual_packages/cuda.py` editiert.
    Aus 

    ```python
    def cached_cuda_version():
        """A cached version of the cuda detection system."""
        return cuda_version()
    ```

    wurde

    ```python
    def cached_cuda_version():
        """A cached version of the cuda detection system."""
        return ""
    ```

    Nun lieferte `conda info` auch wieder die Informationen über conda:

    ```bash
        
    bash-5.0$ conda info

        active environment : None
        user config file : /.condarc
    populated config files :
            conda version : 23.9.0
        conda-build version : not installed
            python version : 3.10.4.final.0
        virtual packages : __archspec=1=x86_64
                            __cuda=0=0
                            __glibc=2.28=0
                            __linux=4.19.0=0
                            __unix=0=0
        base environment : /miniconda3  (writable)
        conda av data dir : /miniconda3/etc/conda
    conda av metadata url : None
            channel URLs : https://repo.anaconda.com/pkgs/main/linux-64
                            https://repo.anaconda.com/pkgs/main/noarch
                            https://repo.anaconda.com/pkgs/r/linux-64
                            https://repo.anaconda.com/pkgs/r/noarch
            package cache : /miniconda3/pkgs
                            /.conda/pkgs
        envs directories : /miniconda3/envs
                            /.conda/envs
                platform : linux-64
                user-agent : conda/23.9.0 requests/2.31.0 CPython/3.10.4 Linux/4.19.0-21-amd64 / glibc/2.28
                    UID:GID : 31358:1003
                netrc file : None
            offline mode : False
    ```

8.  Mit `conda install python=3.12` kann ich nun die aktuelle Python version installieren.
    Auch hier kommt es wieder zur bekannten Fehlermeldung, dieses mal befindet sich die Datei im Pfad `/miniconda3/lib/python3.12/site-packages/conda/plugins/virtual_packages/cuda.py`

9. Nun ist das Ziel erreicht.
    Ich logge mich noch einmal bei SSH aus und ein.
    Nun steht nicht nur (`base`) vor der Eingabezeile, ich kann mir auch via `python --version` die aktuelle Version anzeigen lassen.

    ```bash
    (base) bash-5.0$ python --version
    Python 3.12.2
    ```

10. Für meine einzelnen Webseiten lege ich separate virtuelle Umgebungen an.
    Dies mache ich mit dem Kommando `conda create -n myWebsite python=3.12`.
    Der Wechsel (innerhalb von SSH) erfolgt dann via `conda activate myWebsite`.

11. Natürlich ist in der virtuellen Umgebung auch pip (und andere Packet-Manger) verfügbar)   .
    Damit die Pakete nicht global, sondern nur in der konkreten virtuellen Umgebung installiert werden, verwende ich `python -m pip install <package>`.


# Fazit

Die Installation von Python bietet einige Stolperstellen.
Wenn diese bewältigt sind, erfolgen die weiteren Schritte.
Zum [Erstellen der passanger-wsgi.py Datei für Django]({%post_url 2024-04-05-2024-04-04-netcup Erstellen der passenger-wsgi.py für Django %}) habe ich einen eigenen Beitrag geschrieben.
