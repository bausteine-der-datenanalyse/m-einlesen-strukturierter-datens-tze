**folgende Teilkapitel werden in den w-Python ausgelagert: Datums- und Zeitinformationen in Python, Alles ist relativ: die Epoche, Naive und bewusste Datetime-Objekte, Zeit, Zeitzonen, Kalender [grundlegende Zeittypen](https://pandas.pydata.org/docs/user_guide/timeseries.html#overview)**

### Datums- und Zeitinformationen in Python
Die Verarbeitung von Datums- und Zeitinformationen wird in Python durch verschiedene Module ermöglicht. Im Folgenden wird die Arbeit mit Zeitreihen mit den Modulen NumPy und Pandas weiter ausgeführt. Dennoch sollten Sie die hier aufgezählten Module kennen, da in der Dokumentation gelegentlich auf diese verwiesen wird.

  - Das Modul time stellt Zeit- und Datumsoperationen mit Objekten vom Typ `struct_time` bereit. ([Dokumentation des Moduls time](https://docs.python.org/3/library/time.html))

       - Das Modul datetime führt die Datentypen `datetime` und `timedelta`, zusätzliche Methoden für die Bearbeitung und die Ausgabe von Datums- und Zeitinformationen ein. Das Modul kann Jahreszahlen von 1 bis 9999 nach unserer Zeitrechnung im Gregorianischen Kalender verarbeiten. ([Dokumentation des Moduls datetime](https://docs.python.org/3/library/datetime.html#))
       
       - Das Modul calendar führt verschiedene Kalenderfunktionen ein und erweitert den verarbeitbaren Zeitraum. Basierend auf dem Gregorianischen Kalender reicht dieser in beide Richtungen ins Unendliche. ([Dokumentation des Moduls calendar](https://docs.python.org/3/library/calendar.html#module-calendar))

       - Das Modul pytz führt die IANA-Zeitzonendatenbank (Internet Assigned Numbers Authority) für Anwendungsprogramme und Betriebssysteme ein (auch Olsen-Datenbank genannt). Die IANA-Datenbank beinhaltet die Zeitzonen und Änderungen der Zeit seit 1970. ([Wikipedia](https://de.wikipedia.org/wiki/Zeitzonen-Datenbank)) Das Modul pytz sorgt für eine korrekte Berechnung von Zeiten zum Ende der Zeitumstellung (Ende Sommerzeit) über Zeitzonen hinweg. ([Dokumentation pytz](https://pythonhosted.org/pytz/)) **Hinweis für mich: This library differs from the documented Python API for tzinfo implementations.**

   - NumPy führt die Datentypen `datetime64` und `timedelta64` ein. Diese basieren auf dem Gregorianischen Kalender und reichen in beide Richtungen ins Unendliche. <https://numpy.org/doc/stable/reference/arrays.datetime.html>

  - Pandas nutzt die NumPy-Datentypen `datetime64` und `timedelta64` und ergänzt zahlreiche Funktionen zur Verarbeitung von Datums- und Zeitinformationen aus anderen Paketen. <https://pandas.pydata.org/docs/user_guide/timeseries.html>

NumPy und Pandas können Datetime-Objekte anderer Module in `datetime64` umwandeln.

#### Alles ist relativ: die Epoche
Python speichert Zeit relativ zu einem zeitlichen Bezugspunkt, der Unix-Zeit, der sogenannten Epoche. Die Epoche kann wie folgt mit den verschiedenen Modulen ausgegeben werden.  
**Was fällt Ihnen auf?** (*Nicht gemeint ist `tm_isdst=0`, das angibt, ob daylight saving time, also Sommerzeit, ist.*)

``` {python}
# time
import time as time
print(time.gmtime(0))

# datetime
import datetime as datetime
print("datetime:", datetime.datetime.fromtimestamp(0))

# NumPy
import numpy as np
print("NumPy:", np.datetime64(0, 's'))

# pandas
import pandas as pd
print("Pandas:", pd.to_datetime(0))
```

::: {#tip-aware .callout-tip collapse="true"}
## Lösung Epoche
Das Modul datetime verwendet die Zeitzone Ihres Computers und gibt deshalb den Zeitpunkt der Epoche zur ersten Stunde des Tages zurück (zumindest, wenn auf Ihrem System die zentraleuropäische Zeit UTC+1 eingestellt ist.). Um den gleichen Rückgabewert wie für die anderen Module zu erhalten, muss die Zeitzone angegeben werden.
``` {python}

print(datetime.datetime.fromtimestamp(0))
print(datetime.datetime.fromtimestamp(0, tz = datetime.timezone.utc))
```

:::

#### Naive und bewusste Datetime-Objekte
Datetime-Objekte werden abhängig davon, ob sie Informationen über Zeitzonen enthalten, als naiv (naive) oder als bewusst (aware) bezeichnet. Naiven Datetime-Objekten fehlt diese Information, bewusste Datetime-Objekte enthalten diese. Objekte der Module time, datetime und Pandas verfügen über ein Zeitzonenattribut, sind also bewusst. `np.datetime64` ist seit NumPy-Version 1.11.0 ein naiver Datentyp, unterstützt aber Zeitzonen aus Gründen der Rückwärtskompatibilität.

::: {.border layout="[5, 90, 5]"}

&nbsp;

"Deprecated since version 1.11.0: NumPy does not store timezone information. For backwards compatibility, datetime64 still parses timezone offsets, which it handles by converting to UTC±00:00 (Zulu time). This behaviour is deprecated and will raise an error in the future." [NumPy Dokumentation](https://numpy.org/doc/stable/reference/arrays.datetime.html)  

&nbsp;

:::

::: {#wrn-datetimetimezones .callout-warning appearance="simple" collapse="true"}
## Zeitzonen mit dem Modul datetime
Das Modul datetime unterstützt die Spezifikation von anderen Zeitzonen nicht (sondern nutzt die Angabe einer Zeitdifferenz, timedelta). Um eine andere Zeitzone anzugeben, wird das Modul pytz benötigt. Dabei gilt es, eine Besonderheit zu beachten.

``` {python}

import pytz
moscow_tz = pytz.timezone('Europe/Moscow')
print(datetime.datetime.now(moscow_tz))

# Vorsicht mit der Zeitzonenangabe via Etc/UTC
## Der korrekte Zeitzonenversatz (zur Winterzeit)
moscow_tz = pytz.timezone('Etc/GMT+3')
print(datetime.datetime.now(moscow_tz))

## Zeitzonenversatz mit invertiertem Vorzeichen
moscow_tz = pytz.timezone('Etc/GMT-3')
print(datetime.datetime.now(moscow_tz))
```

:::: {.border layout="[5, 90, 5]"}

&nbsp;

"Das Spezialgebiet 'Etc' wird für einige administrative Zonen verwendet, insbesondere für 'Etc/UTC', die koordinierte Weltzeit. Für die Kompatibilität mit dem POSIX-Format haben die Zonen, welche mit 'Etc/GMT' beginnen, genau das gegenteilige Vorzeichen als man erwarten würde. In diesem Format haben alle Zonen westlich der GMT ein positives Vorzeichen und alle östlich davon ein negatives." ([Wikipedia](https://de.wikipedia.org/wiki/Zeitzonen-Datenbank))

&nbsp;

::::

Alle verfügbaren Zeitzonen können Sie sich wie folgt ausgeben lassen (hier ohne Ausgabe).

``` {python}
#| output: false

from zoneinfo import available_timezones
for timezone in sorted(available_timezones()):
  print(timezone)
```

:::

#### Zeit
NumPy nutzt die Internationale Atomzeit (abgekürzt TAI für französisch Temps Atomique International). Diese nimmt für jeden Kalendertag eine Länge von 86.400 Sekunden an, kennt also keine Schaltsekunde. Die Atomzeit bildet die Grundlage für die koordinierte Weltzeit UTC. UTC steht für Coordinated Universal Time (auch bekannt als Greenwich Mean Time). Das Kürzel UTC ist ein Kompromiss für die englische und die französische Sprache. Die koordinierte Weltzeit gleicht die Verlangsamung der Erdrotation (astronomisch gemessen als Universalzeit, Universal Time UT) durch Schaltsekunden aus, um die geringfügige Verlängerung eines Tages auzugleichen. Die TAI geht deshalb gegenüber der UTC vor. Seit 1972 unterscheiden sich beide Zeiten um eine ganzzahlige Anzahl von Sekunden. Aktuell (2024) geht die TAI 37 Sekunden gegenüber UTC vor.
Eine Umwandlung in die koordinierte Weltzeit ist in NumPy bislang noch nicht umgesetzt. ([Dokumentation NumPy](https://numpy.org/doc/stable/reference/arrays.datetime.html), [Wikipedia](https://de.wikipedia.org/wiki/Internationale_Atomzeit))
(**wie ist das in Pandas?! Der Absatz ist fast wörtlich übernommen.**) 

#### Zeitzonen
**ergänzen** <https://de.wikipedia.org/wiki/Zeitzone>

**Können wir die mit Pandas plotten?**

#### Dailight Saving Time
::: {.border layout="[5, 90, 5]"}

&nbsp;

"DST is Daylight Saving Time, an adjustment of the timezone by (usually) one hour during part of the year. DST rules are magic (determined by local law) and can change from year to year. The C library has a table containing the local rules (often it is read from a system file for flexibility) and is the only source of True Wisdom in this respect."

&nbsp;
:::

**Können wir die Länder mit Zeitumstellung mit Pandas plotten?**

#### Kalender
Der von den Modulen calendar, NumPy (und Pandas?!) verwendete, vor die Zeit seiner Einführung 1582 erweiterte Gregorianische Kalender heißt [propleptischer Gregorianischer Kalender](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar)

Schaltjahre **im w-Python kleines Programm schreiben lassen, dass auf Schaltjahre prüft**

Welche wichtigen Kalender gibt es (jüdischen Kalender, muslimischen Kalender, chinesischen Kalender)

Mögliche Aufgabe: Kalender hin und her konvertieren

