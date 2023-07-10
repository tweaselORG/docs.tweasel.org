{
    "title": "Installation und Einrichtung",
    "weight": 10,
    "description": "In diesem Artikel wird erklärt, wie die Tweasel-Bibliotheken und -Tools installiert und die Umgebung für die Analyse von mobilen Apps eingerichtet werden kann. Außerdem wird gezeigt, wie die Geräte und Emulatoren auf Android und iOS vorbereitet werden."
}

## Installation der Bibliotheken und Tools

Die [Bibliotheken und Tools des Tweasel-Projekts](/) werden als NPM-Pakete bereitgestellt. Sie benötigen [Node.js](https://nodejs.org) (mindestens Version 18) und können über NPM oder Yarn installiert werden. Alle anderen Abhängigkeiten (wie z. B. das Android-SDK, Frida und pymobiledevice3) werden dadurch automatisch installiert.

Node.js, NPM und das Tweasel-CLI können unter Ubuntu 23.04[^ubuntu-node] beispielsweise wie folgt installiert werden:

[^ubuntu-node]: Achtung: Die Paketquellen älterer Ubuntu-Versionen enthalten zu alte Versionen von Node.js. Hier muss Node.js daher auf anderem Wege installiert werden, etwa über [nvm](https://github.com/nvm-sh/nvm) oder die [NodeSource-Pakete](https://github.com/nodesource/distributions).

```sh
sudo apt update
sudo apt install nodejs npm

sudo npm i -g tweasel-cli
```

Die Bibliotheken hingegen sollten lokal in einem Projekt (also ohne `-g`) installiert werden, etwa für cyanoacrylate:

```sh
npm i cyanoacrylate
```

### Einrichtung für physische Geräte

Soll mit physischen Geräten gearbeitet werden, ist zusätzlich etwas manuelle Einrichtung nötig, abhängig von der Plattform.

Die Einrichtung für Android ist in der [Android-Developer-Dokumentation](https://developer.android.com/studio/run/device#setting-up) beschrieben. Unter Ubuntu muss die Nutzer_in Mitglied der `plugdev`-Gruppe sein (`sudo usermod -aG plugdev <Nutzer_innenname>`) und es müssen `udev`-Regeln für das Gerät installiert sein (`sudo apt install android-sdk-platform-tools-common`). Tipps für andere Linux-Distributionen finden sich bei [`android-udev-rules`](https://github.com/M0Rf30/android-udev-rules).

Für iOS sind nur unter Windows zusätzliche Einrichtungsschritte nötig. Hier muss der Apple-Device-Driver und der Apple-Application-Support installiert werden. Diese werden zusammen mit iTunes installiert.

## Gerätevorbereitung

Unsere Bibliotheken funktionieren mit physischen Handys und Emulatoren unter Android und physischen Handys unter iOS. Wir testen in den folgenden Umgebungen (andere Versionen funktionieren aber möglicherweise auch):

| Plattform | Art | Getestete Versionen |
| --- | --- | --- |
| Android | `device` (Moto G7 Power) | 13 (API-Level 33) |
| Android | `emulator` | 11 (API level 30), 13 (API-Level 33) |
| iOS | `device` (iPhone X, iPhone 6S) | 15.6.1, 16.0 |

Je nachdem, was für ein Gerät für die Analyse verwendet werden soll, ist die unten beschriebene Vorbereitung nötig. Das Installieren und Einrichten aller weiteren Abhängigkeiten auf dem Gerät übernehmen unsere Tools automatisch.

### Physische Android-Handys

Auf physischen Android-Handys muss USB-Debugging aktiviert werden. Das geht über *Einstellungen* -> *System* -> *Entwickleroptionen* (zum Aktivieren sieben Mal auf die *Build-Nummer* unter *Einstellungen* -> *Über das Telefon* -> *Android-Version* tippen) -> *USB-Debugging*.

Für sinnvolle Trafficanalysen muss das Gerät darüber hinaus gerootet sein. Dafür empfehlen wir [Magisk](https://topjohnwu.github.io/Magisk/). Dafür muss auch der Bootloader entsperrt werden. Wie das geht, variiert von Gerät zu Gerät. In der Regel gehen dabei alle Daten auf dem Gerät verloren.  
Nach dem Rooten sollte, falls verfügbar, Root-Debugging aktiviert werden: *Einstellungen* -> *System* -> *Entwickleroptionen* -> *Root-Debugging*.

Bei der Verbindung per USB muss bestätigt werden, dass dem Rechner vertraut wird.

### Android-Emulator

Zur Nutzung eines Android-Emulators ist keine besondere Einrichtung nötig. Es muss nur ein Emulator erstellt werden. Das geht über den Device Manager von [Android Studio](https://developer.android.com/studio) oder über das Tweasel-CLI, es empfiehlt sich dabei, einen größeren Speicherplatz einzustellen:

```sh
tweasel android-emulator:create "<Name des Emulators>" --partition-size 16384
```

Das erste Ausführen kann einen Moment dauern, da im Hintergrund automatisch das Android-SDK eingerichtet wrid. Der Befehl fragt dann interaktiv ab, was für ein Emulator erstellt werden. Für die größtmögliche Kompatibilität mit Apps [empfehlen wir aktuell](https://github.com/tweaselORG/appstraction/issues/54): Android 11 (API-Level 30), Android mit Google APIs, x86_64.

Soll der Emulator noch weiter eingerichtet werden (etwa um Honey-Data zu platzieren), kann er wie folgt gestartet und anschließend ein Snapshot erstellt werden:

```sh
tweasel android-emulator:start "<Name des Emulators>"
tweasel android-emulator:snapshot:create "<Name des Snapshots>" 
```

### Physische iPhones

Auch iOS-Geräte benötigen einen Jailbreak. Getestet sind unsere Tools auf iOS 15 und 16 mit dem [palera1n-Jailbreak](https://github.com/palera1n/palera1n). Dazu [dieser Anleitung](https://ios.cfw.guide/installing-palera1n/) folgen. Wichtig: Der Jailbreak muss im Rootful-Modus installiert werden.[^ios-14]

[^ios-14]: In anderen Projekten haben wir früher erfolgreich iOS 14 mit dem [checkra1n-Jailbreak](https://checkra.in/) verwendet. Da wir aber kein Gerät mit iOS 14 mehr haben, können wir nicht garantieren, dass das auch mit den Tweasel-Tools funktioniert.

Nach dem Jailbreak muss über Sileo das Paket `openssh-server` installiert werden.

Bei der Verbindung per USB muss bestätigt werden, dass dem Rechner vertraut wird.

Zur optimalen Nutzung (insbesondere zur Reduzierung von Hintergrundtraffic des Systems und anderer Apps) empfehlen wir zusätzlich die folgenden Einstellungen zu setzen, dies ist jedoch optional:

* General
    * Background App Refresh: aus
    * Software Update
        * Automatic Updates: aus
* Display & Brightness
    * Auto-Lock: never
* Privacy & Security
    * Location Services: an
    * Analytics & Improvements
        * Share iPhone Analytics: aus
* App Store
    * Automatic Downloads
        * Apps: aus
        * App Updates: aus
