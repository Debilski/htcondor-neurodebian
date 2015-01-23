Voraussetzungen
---------------

  - Beliebig viele Rechner, die zu einem Rechencluster zusammen gebunden werden sollen
  - Eine virtuelle Maschine, die als Condor Host dienen soll. (Erreichbarkeit sollte gut sein, Maschine darf aber zwischendurch auch mal rebooten.)
  - Nutzer:Gruppe condor:condor im LDAP mit
        $ id condor
        uid=1111(condor) gid=1111(condor) groups=1111(condor)

Hinzufügen der Neurodebian-Quellen
----------------------------------

Auf http://neuro.debian.net/ nachsehen, welcher Server gebraucht wird und dann beispielsweise:

    $ wget -O- http://neuro.debian.net/lists/wheezy.de-md.full | sudo tee /etc/apt/sources.list.d/neurodebian.sources.list
    $ sudo apt-key adv --recv-keys --keyserver hkp://pgp.mit.edu:80 2649A5A9

ausführen, um die Quellen hinzuzufügen.

Systeme updaten mit

    $ aptitude update

Condor installieren
-------------------

    $ aptitude install htcondor

Condor konfigurieren (clients)
------------------------------

Datei in `/etc/condor/config.d/01local` anlegen:

    NETWORK_INTERFACE = <% IP-Adresse des Rechners %>
    CONDOR_HOST = <% IP-Adresse des Hosts %>
    CONDOR_ADMIN = <% E-Mail-Adresse %>
    CONDOR_IDS = 1111.1111
    COLLECTOR_NAME = <% Name des Clusters %>
    UID_DOMAIN = <% Domain %>
    FILESYSTEM_DOMAIN = <% Domain %>
    EMAIL_DOMAIN = <% Domain %>
    ALLOW_READ = *
    ALLOW_WRITE = *
    WANT_SUSPEND = FALSE

    PREEMPT = $(UWCS_PREEMPT)
    PREEMPTION_REQUIREMENTS = $(UWCS_PREEMPTION_REQUIREMENTS)
    PREEMPTION_RANK         = $(UWCS_PREEMPTION_RANK)

    DAEMON_LIST = MASTER, SCHEDD, STARTD
    TRUST_UID_DOMAIN = TRUE

    IsDesktop = False
    STARTD_ATTRS = $(STARTD_ATTRS) IsDesktop


Condor konfigurieren (host)
---------------------------

    NETWORK_INTERFACE = <% IP-Adresse des Rechners %>
    CONDOR_HOST = <% IP-Adresse des Hosts %>
    CONDOR_ADMIN = <% E-Mail-Adresse %>
    CONDOR_IDS = 1111.1111
    COLLECTOR_NAME = <% Name des Clusters %>
    UID_DOMAIN = <% Domain %>
    FILESYSTEM_DOMAIN = <% Domain %>
    EMAIL_DOMAIN = <% Domain %>
    ALLOW_READ = *
    ALLOW_WRITE = *
    WANT_SUSPEND = FALSE

    PREEMPT     = $(UWCS_PREEMPT)
    PREEMPTION_REQUIREMENTS     = $(UWCS_PREEMPTION_REQUIREMENTS)
    PREEMPTION_RANK             = $(UWCS_PREEMPTION_RANK)

    DAEMON_LIST = COLLECTOR, MASTER, NEGOTIATOR
    TRUST_UID_DOMAIN = TRUE

Startup-Fixes
-------------

Wenn Concurrent-Boot aktiviert ist, hat Condor Probleme, auf das SigInt-Signal zu reagieren. Dafür muss man gegebenenfalls in der `/etc/init.d/rc` die Zeile

    CONCURRENCY=makefile

durch

    CONCURRENCY="none"

ersetzen, außerdem muss die Datei `/etc/init.d/.legacy-bootordering` entfernt werden (falls vorhanden).

Schließlich muss man noch eine Datei `etc/rc.local` anlegen, um Condor nach dem Booten nochmals neu zu starten:

    #!/bin/sh -e
    #
    # rc.local
    #
    # This script is executed at the end of each multiuser runlevel.
    # Make sure that the script will "exit 0" on success or any other
    # value on error.
    #
    # In order to enable or disable this script just change the execution
    # bits.
    #
    # By default this script does nothing.
    service condor restart
    exit 0

Es empfielt sich jedoch, das System zunächst einmal ohne die Fixes zu testen; es ist möglich, dass diese bereits behoben wurden in aktuellen Versionen.


