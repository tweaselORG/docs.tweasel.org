{
    "title": "Traffic-Analyse mit dem CLI",
    "weight": 20,
    "description": "Mit dem Tweasel-Kommandozeilenwerkzeug kann der Netzwerktraffic von mobilen Apps aufgezeichnet werden und Trafficaufzeichnungen im HAR-Format können auf die Übertragung von Tracking-Daten untersucht werden."
}

Das Tweasel-CLI ermöglicht es, die Funktionen der verschiedenen Bibliotheken des Tweasel-Projekts über die Kommandozeile zu nutzen, ohne eigenen Code schreiben zu müssen.

Die Funktionen des CLIs sind in verschiedene Befehle gruppiert. Die wichtigsten sind:

* `tweasel record-traffic` zum Aufzeichnen von App-Traffic im HAR-Format
* `tweasel detect-tracking` zum Analysieren von Traffic im HAR-Format auf übertragene Tracking-Daten
* `tweasel android-emulator` zum Erstellen, Starten, und Löschen von Android-Emulatoren

Alle Befehle verfügen über umfangreiche Hilfeseiten, die über `--help` angezeigt werden können, also zum Beispiel:

```sh
tweasel --help
tweasel record-traffic --help
```

Der Befehl `tweasel autocomplete` erklärt darüber hinaus, wie sich eine Autovervollständigung für alle Befehle einrichten lässt.

## Traffic aufzeichnen

Zum Aufzeichnen des Traffics einer App gibt es zwei Möglichkeiten. Ist die App bereits auf dem Handy installiert, muss nur die Plattform angegeben werden:

```sh
tweasel record-traffic --platform android
```

Das CLI zeigt dann eine Liste der installierten Apps an und fragt, für welche der Traffic aufgezeichnet werden soll. Alternativ kann die App-ID auch direkt angegeben werden, so z. B. für die Otto-App auf Android:

```sh
tweasel record-traffic de.cellular.ottohybrid --platform android
```

Ist die App noch nicht installiert, kann stattdessen ihre Installationsdatei angegeben werden, die Plattform wird daraus dann automatisch abgeleitet:

```sh
tweasel record-traffic db-navigator.apk
```

Auf iOS werden IPA-Dateien unterstützt. Auf Android werden hier neben einzelnen APK-Dateien auch unterstützt: Split-APKs, zusätzliche OBB-Dateien, XAPK-Dateien von APKPure, APKM-Dateien von APKMirror, und APKS-Dateien von SAI.

Soll ein (bereits existierender) Android-Emulator gestartet werden, geht das wie folgt:

```sh
tweasel record-traffic airbnb.apk --run-target emulator --emulator-name "<Name des Emulators>"
```

Für iOS müssen zwei zusätzliche Argumente angegeben werden, die IP-Adresse des Handys und des Rechners, auf dem das Tweasel-CLI läuft, z. B.:

```sh
tweasel record-traffic check24.ipa --ios-ip 10.0.0.3 --ios-proxy-ip 10.0.0.2
```

Der Traffic wird solange aufgezeichnet, bis Enter gedrückt wird. Soll stattdessen für eine feste Dauer aufgezeichnet werden, geht das mit `--timeout <Zeit in Sekunden>`. Der aufgezeichnete Traffic wird in beiden Fällen standardmäßig als HAR-Datei im aktuellen Verzeichnis gespeichert. Das kann mit `-o` angepasst werden.  
Es besteht auch die Möglichkeit, unterschiedliche „Phasen“ des Traffics in eigene Dateien zu speichern. Das geht mit `--multiple-collections`. Dann wird nach jedem Enter-Drücken gefragt, ob eine weitere Phase aufgezeichnet werden und, wenn ja, wie diese heißen soll. Diese Funktion ist beispielsweise hilfreich, um den Traffic vor und nach der Interaktion mit einem Einwilligungsdialog zu unterscheiden.

Auf Wunsch kann die App nach der Analyse automatisch deinstalliert werden. Das geht mit `--uninstall-app`.

Zu beachten:

* Bei Android wird standardmäßig nur der Traffic der jeweiligen App aufgezeichnet (also insbesondere kein Hintergrundrauschen des Systems und anderer Apps). Soll stattdessen der gesamte Systemtraffic aufgezeichnet werden, geht das mit der `--all-traffic`-Option.  
  Bei iOS hingegen kann aktuell nur der gesamte Traffic des Systems aufgezeichnet werden.
* Standardmäßig werden den Apps alle Berechtigungen gewährt. Das lässt sich bei Bedarf mit `--no-grant-permissions` unterbinden.
* Es gibt noch einige weitere Optionen, die hier nicht aufgeführt sind. Details dazu gibt es mit `--help`.

## Tracking erkennen

Der `detect-tracking`-Befehl hilft, übertragene Daten an Trackingunternehmen in HAR-Traffic-Aufzeichnungen (unabhängig, ob mit unseren oder anderen Tools aufgenommen) zu erkennen. Dazu wird einfach die jeweilige HAR-Datei übergeben:

```sh
tweasel detect-tracking "<Pfad zur HAR-Datei>"
```

Die Erkennung basiert auf Adaptern für spezifische Tracking-Endpunkte (mehr zur Methodik in der [README von TrackHAR](https://github.com/tweaselORG/TrackHAR)). Angezeigt wird für jede Anfrage in der HAR-Datei, für die ein Adapter vorhanden war, eine Tabelle der erkannten übertragenen Daten. Dabei sind die folgenden Angaben für jedes Datum:

* `property`: um welchen Datentyp es sich handelt (z. B. `idfa`: Werbe-ID des Handys, `osName`: Name des Betriebssystems, `appVersion`: Version der App)
* `context`: Teil der Anfrage, in dem das Datum gefunden wurde (z. B. Header, Body)
* `path`: wo in dem jeweiligen Kontext das Datum gefunden wurde
* `value`: das konkret übertragene Datum

Mit `-x` kann zusätzlich eine Begründung für die Zuordnung (`reasoning`) angezeigt werden. Bei jeder Anfrage befindet sich zudem ein Link auf den jeweiligen Adapter in unserem [Tracker-Wiki](https://trackers.tweasel.org/). Dort finden sich noch weitere technische Details zum Tracking-Endpunkt und dazu, wie unsere Dekodierung funktioniert.

{{< hint warning >}}
Wichtig zu bedenken ist, dass das adapterbasierte Verfahren immer nur eine untere Schranke liefern kann: Die angezeigten Daten wurden definitiv übertragen, aber es ist gut möglich (sogar sehr wahrscheinlich), dass weitere übertragene Daten nicht erkannt wurden.
{{< /hint >}}

Deshalb kann zusätzlich mit Honey-Data und Indikatoren gearbeitet werden. Dabei wird dem `detect-tracking`-Befehl eine Aufstellung von bekannten Attributen des Geräts mitgegeben als JSON-Objekt, wie etwa hier:

```sh
tweasel detect-tracking "<Pfad zur HAR-Datei>" --indicators '{ "localIp": ["10.0.0.2", "fd31:4159::a2a1"], "idfa": "6a1c1487-a0af-4223-b142-a0f4621d0311" }'
```

Hier wurden die lokalen IP-Adressen des Handys und die Werbe-ID angegeben. In den Anfragen, die von keinem Adapter abgedeckt wurden, wird jetzt nach diesen gesucht. Dabei werden nicht nur Plain-Text-Übertragungen erkannt, sondern auch Base64- und URL-enkodierte. Die dadurch erkannten Übertragungen werden auch in der Tabelle angezeigt und separat gekennzeichnet.

Aber auch hierbei ist zu beachten, dass immer noch Daten übersehen werden können.

Die Bezeichnungen für die Werte (im Beispiel `localIp` und `idfa` können frei gewählt werden). Statt das JSON-Objekt direkt im Befehl anzugeben, kann auch eine JSON-Datei mit den Indikatoren angelegt und deren Pfad übergeben werden:

```sh
tweasel detect-tracking "<Pfad zur HAR-Datei>" --indicators "<Pfad zur JSON-Datei>"
```
