{
    "title": "Einführung",
    "description": "Tweasel ist ein Projekt, das Infrastruktur für die Erkennung von und das Beschweren über Tracking und Datenschutzverletzungen in mobilen Apps auf Android und iOS baut. Diese Seite gibt einen Überblick der Werkzeuge und Bibliotheken, die Teil des Projekts sind."
}

Tweasel ist ein Projekt, das Infrastruktur für die Erkennung von und das Beschweren über Tracking und Datenschutzverletzungen in mobilen Apps auf Android und iOS baut.

Als Teil des Projekts haben wir Werkzeuge für die automatisierte App-Analyse und Tracking-Erkennung entwickelt. Diese Dokumentation erklärt, wie die Werkzeuge funktionieren, wie die Ergebnisse verifiziert und ohne unsere Werkzeuge nachvollzogen werden können. Diese Seite gibt einen Überblick der Werzeuge und ihrer Fähgkeiten. Der Code für alle unsere Projekt liegt [auf GitHub](https://github.com/orgs/tweaselORG/repositories).

Das Projekt wird von einem gemeinnützigen Datenschutzverein, Datenanfragen.de e.&thinsp;V., betrieben, der unter anderem auch die Webseite [datenanfragen.de](https://www.datenanfragen.de) betreibt. Es [erhält Mittel](https://nlnet.nl/project/TrackingWeasel/) aus dem Zero Entrust Fund im Next Generation Internet Programm der Europäischen Kommission über die nlnet Stiftung.

Hier ist die Übersetzung auf Deutsch. Ich habe versucht, mit Unterstrich zu gendern und direkte Ansprachen zu vermeiden:

## Appstraction

[Appstraction](https://github.com/tweaselORG/appstraction) bietet eine Abstraktionsebene für häufige Instrumentierungsfunktionen auf mobilen Plattformen, insbesondere Android und iOS. Dazu gehören das Installieren, Deinstallieren und Starten von Apps, das Verwalten ihrer Berechtigungen, aber auch das Verwalten von Geräten, wie das Zurücksetzen auf Snapshots, das Festlegen des Zwischenablageinhalts, das Konfigurieren des Proxys usw. Appstraction ist hauptsächlich für den Einsatz in der mobilen Datenschutzforschung konzipiert, kann aber auch für andere Zwecke verwendet werden. Es handelt sich um eine Bibliothek, daher muss ein Skript geschrieben werden, um die Funktionen zu nutzen.

## Cyanoacrylate

[Cyanoacrylate](https://github.com/tweaselORG/cyanoacrylate) ist der Klebstoff, der `appstraction` mit anderen Tools wie `mitmproxy` verbindet, um ein einfaches Toolkit zu erstellen, mit dem man ohne viel Benutzer_inneninteraktion Traffic-Analysen für viele Apps durchführen kann. Dies ist vor allem dazu gedacht, das Tracking-Verhalten von mobilen Apps zu analysieren. Es unterstützt das Ausführen von Apps auf Android und iOS, derzeit auf physischen Geräten sowie einem Emulator für Android.

Unter anderem kann es:

- Überprüfen, ob der Geräte-Traffic durch einen DNS-Blocker verändert wird
- Einen Android-Emulator starten und verwalten
- HTTP(S)-Traffic im HAR-Format sammeln
- Automatisch Certificate Authorities und das WireGuard-mitmproxy-Setup verwalten

Es ist auch als Bibliothek geschrieben. Für jede Analyse muss ein benutzerdefiniertes Skript geschrieben werden, das den spezifischen Anforderungen entspricht.

## TrackHAR

[TrackHAR](https://github.com/tweaselORG/TrackHAR) ist eine Bibliothek zum Erkennen von Tracking-Datenübertragungen aus Traffic im HAR-Format. Es verwendet zwei Ansätze, um Tracking-Anfragen zu identifizieren:

- Adapterbasiertes Parsen, das sogenannte Adapter verwendet, die für spezifische Arten von Anfragen (z.B. Anfragen, die an einen bestimmten Endpunkt gehen) geschrieben wurden. Diese Adapter definieren Algorithmen zum Dekodieren der Anfrage, die oft mit mehreren verschachtelten Kodierungen verschlüsselt sind, und ordnen den dekodierten Inhalt vordefinierten Datenkategorien zu, um zu identifizieren, welche Daten übertragen wurden. Die Adapter sind in [dem Tracker-Wiki](https://trackers.tweasel.org) dokumentiert.
- Indikator-Matching, das nach gegebenen Zeichenketten - Indikatoren - in den Anfragen sucht, um Honey-Data zu identifizieren, die zuvor auf dem Gerät platziert wurden. TrackHAR verwendet dies als Alternative, wenn keine Adapter verfügbar sind, um eine Anfrage zuzuordnen. Derzeit unterstützt es auch das Matching von base64- oder URL-kodierten Indikatoren.

## CLI

Das [tweasel CLI](https://github.com/tweaselORG/cli) ist ein Kommandozeilenprogramm, das `TrackHAR`, `cyanoacrylate` und andere Hilfsprogramme kombiniert, um ein einzelnes Tool für manuelle mobile Tracking-Analysen zu erstellen. Es bietet eine Schnittstelle zum Starten und Automatisieren der Traffic-Aufzeichnung, zum Verwalten von Android-Emulatoren und zum Erkennen von Tracking in den aufgezeichneten Anfragen. Für den Einstieg in das Projekt eignet sich das CLI gut, um die Tools auszuprobieren.

## parse-tunes/parse-play

[parse-tunes](https://github.com/tweaselORG/parse-tunes) und [parse-play](https://github.com/baltpeter/parse-play) sind zwei Bibliotheken zum Parsen von Daten aus dem Apple App Store und dem Google Play Store. Sie können damit die App-Ranking-Charts abrufen oder auf die Datenschutzinformationen zugreifen, die die App-Entwickler_innen dem Store zur Verfügung gestellt haben.
