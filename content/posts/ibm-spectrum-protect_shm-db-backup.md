+++
title = "Spectrum Protect - Backup über Shared Memory"
description = ""
type = ["posts","post"]
tags = [
    "IBM Spectrum Protect",
    "Backup",
    "Performance"
]
date = "2020-07-08"
categories = [
    "IBM Spectrum Protect",
]
series = ["IBM Spectrum Protect"]
[ author ]
  name = "Jan Hölscher"
+++

IBM Spectrum Protect verwendet für das Backup der Datenbank normalerweise eine Verbindung über TCPIP. Hier soll gezeigt werden, wie die Verbindung auf Shared Memory umgestellt werden kann. Die Änderungen müssen zum einene in der Serverinstanz und zum anderen in einer speziellen `dsm.sys` erfolgen.

{{% notice note %}}
Wurde mit der Version Tivoli Storage Manger 7.1 eingeführt.
{{% /notice %}}

## Server
Die Serverinstanz wird mittels der Datei `dsmserv.opt` im Home Verzeichnis des Instanzusers konfiguriert. 

Dort müssen die folgenden Zeilen hinzugefügt werden: 
```
COMMMethod  SHAREdmem
SHMPort 1510
```

Damit die Änderungen wirksam werden muss die Serverinstanz neugestartet werden. 

Im Anschluss kann mittels Telnet überprüft werden, ob der Port wirklich geöffnet und erreichbar ist. 
```
# telnet localhost 1510
Trying... 
Connected to localhost.
Escape character is '^]'.
```

Bis jetzt ist allerdings nur der Kommunikationsweg `SHAREdmem` geöffnet, das Database Backup läuft weiterhin über TCPIP. 

## Database Backup Client
Die Verbindung vom speziellen Node `$$_TSMDBMGR_$$` wird, wie für einen normalen Backup-Archive Client, in einer `dsm.sys` Datei gespeichert. 

Die spezielle Datei ist unter den folgenden Pfaden zu finden: 
```
# AIX 
/opt/tivoli/tsm/server/bin/dbbkapi/dsm.sys
```

Diese sollte folgenden Inhalt aufweisen:
```
COMMMethod      SHAREdmem
SHMPort         1510
NODENAME        $$_TSMDBMGR_$$
PASSWORDDIR     <HOME vom Instanzuser>
ERRORLOGNAME    <HOME vom Instanzuser>/tsmdbmgr.log
```

## Performance
In meinem Test Setup sind die Backups der Datenbanken ungefähr mit dem Faktor 2x schneller, zudem ist die Performance durchgängiger. D.h. es gibt derzeit fast keinen Ausnahmen, bei denen ein Datenbank Backup aufgund hoher Systemauslastung zu lange braucht. 