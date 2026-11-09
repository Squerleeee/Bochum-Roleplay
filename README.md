# Bochum Roleplay

Eigenständiges FiveM-Grundframework für Bochum Roleplay. Es baut nicht auf ESX oder QBCore auf.

## Voraussetzungen

- FXServer-Artefakte und eine gültige FiveM-Lizenz
- MySQL/MariaDB
- [oxmysql](https://github.com/overextended/oxmysql) als Datenbanktreiber (keine ESX-/QBCore-Abhängigkeit)

FiveM stellt in Lua keinen nativen MySQL-Treiber bereit. oxmysql ist deshalb die einzige externe Resource, die der BO-Kern benötigt. Bei manueller Installation muss die Datenbank `bo_framework` vor dem Start angelegt werden; Spieler-, Immobilien-, Schlüssel-, Fahrzeug- und Anti-Cheat-Tabellen werden beim Start automatisch erzeugt. Das txAdmin-Recipe legt die Datenbank im Deployer-Schritt an und verwendet dessen Verbindungszeichenfolge.

## Start

1. Lege `bo_core` unter `resources/[bo]/bo_core` ab.
2. Installiere oxmysql als Resource und trage in `server.cfg` Lizenzschlüssel sowie MySQL-Verbindungsdaten ein.
3. Erstelle die Datenbank, z. B. `CREATE DATABASE bo_framework CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`.
4. Starte den Server mit `server.cfg`.
5. Weise Adminrechten ausschließlich vertrauenswürdigen Konten zu: `add_ace identifier.license:DEINE_LICENSE bo.admin allow`.

Die Beispielkonfiguration enthält absichtlich Platzhalter und ist vor dem öffentlichen Betrieb anzupassen.

### Installation mit txAdmin

Das Recipe liegt unter `txadmin-recipe.yaml`. Vor dem Import muss der Ordner `resources/[bo]` in einem öffentlichen GitHub-Repository liegen: txAdmin lädt diesen Ordner zurzeit von `https://github.com/YOUR_GITHUB_USERNAME/bochum-roleplay` herunter. Erstelle auf GitHub ein öffentliches Repository namens `bochum-roleplay`, lade den Projektordner mit `.gitignore` und `resources/[bo]` hoch und ändere in `txadmin-recipe.yaml` `YOUR_GITHUB_USERNAME` zu deinem GitHub-Benutzernamen. Importiere danach die Recipe-Datei über eine erreichbare HTTPS-URL in txAdmin. Im Database-Schritt gibst du die oxmysql-Verbindung an. Das Recipe verwendet die offizielle oxmysql-Version 2.14.3 und eine festgelegte Revision der Cfx-System-Resources. Lizenzschlüssel und Datenbankzugangsdaten gehören nicht ins öffentliche Repository.

Vor dem Upload prüfen: `.gitignore` schließt `server.cfg`, `.env`, Logs, Cache und Node-Abhängigkeiten aus. Veröffentliche niemals echte Lizenzschlüssel, Datenbankpasswörter, Discord-Bot-Tokens oder Webhook-URLs. Die manuelle lokale `server.cfg` wird nicht vom Recipe heruntergeladen; txAdmin schreibt eine neue Konfiguration mit den beim Setup eingegebenen Werten.

Die hier markierte `server.cfg` ist die manuelle Starterkonfiguration. Stelle Lizenzschlüssel und Datenbankverbindung vor einem manuellen Serverstart auf deine eigenen Werte ein. txAdmin erzeugt beim Recipe-Import eine eigene `server.cfg` aus seinen Eingabewerten.

## Enthalten

- **Eigenständige Resources (erster Ausbau-Schritt):** `bo_core`, `bo_survival`, `bo_ems` und `bo_police`. Weitere eigenständige Resources sind noch nicht umgesetzt; der gewünschte Umfang von 200 ist ausdrücklich nicht erreicht.
- persistentes Hunger-/Durstsystem mit HUD; Brot und Wasser aus dem BO-Inventar füllen die Werte auf
- Rettungsdienst mit `/911 <Nachricht>`, `/revive <Spieler-ID>` und `/heal <Spieler-ID>`; Behandlung und Wiederbelebung sind auf den Rettungsdienst und nahe Ziele begrenzt
- Polizeifesseln mit `/cuff <Spieler-ID>` und `/uncuff <Spieler-ID>`; Berechtigung und 3-Meter-Abstand werden serverseitig geprüft
- Spielerprofile mit persistentem Namen, Beruf, Rang, Bargeld und Bankguthaben
- automatischer Erststart-Charakter und Spawn
- öffentlich wählbare Berufe (`/jobs`, `/jobjoin <name>`) und standortgebundene Arbeitsaufträge (`/work`, 60 Sekunden Abklingzeit)
- Bargeldzahlung (`/pay <Spieler-ID> <Betrag>`)
- dunkles NUI-Dashboard (F1 oder `/bo`) mit Kontostand, Überweisungen und Berufswahl
- kaufbare, zugangsbeschränkte Häuser mit festgelegten Außen- und Innenpositionen (`/houses`, `/leavehouse`)
- persistente Hauszugänge mit serverseitig geprüfter Schlüsselvergabe an Online-Spieler und widerrufbaren Schlüsseln
- Fahrzeughändler und persistente Eigentümergaragen mit Kennzeichen, serverseitiger Besitzprüfung sowie Ein-/Ausparken (`/garage`, `/park`)
- Inventar mit Shop, Wasser, Brot und Erste-Hilfe-Set (`/inventory`)
- Telefonkontakte und persistente Nachrichten an gespeicherte Kontakte (`/phone`)
- polizeiliche Personensuche und Ermittlungsberichte (`/mdt`, nur mit Polizeiberuf)
- servergespeicherte mobile Blitzer (`/blitzer`, `/blitzerentfernen`), Geschwindigkeitsmessung und bezahlbare Bußgelder (`/bußgeld`)
- ACE-geschützte Adminübersicht, Berufszuteilung, Barauszahlung und Kick (`/boadmin`)
- serverseitiges Anti-Cheat mit Bewegungs-Baseline, Teleport-/Speed-/Noclip-Heuristiken, Weltgrenzenprüfung und konfigurierbaren überwachten Zonen
- persistente Anti-Cheat-Protokolle (`/boac [Profil-ID]`, ACE-geschützt)
- periodisches Speichern und Speichern beim Verlassen

## Anti-Cheat-Hinweise

Verdachtsfälle werden in `bo_ac_incidents` gespeichert und online befindlichen ACE-Admins angezeigt. Serverseitige Bewegungsprüfungen benötigen OneSync. Prüfintervall, Bewegungsschwellen und überwachte Koordinaten/Zonen können unter `BOConfig.AntiCheat` angepasst werden.

Ein permanenter automatischer Bann wird standardmäßig erst bei mindestens **14 Punkten aus mindestens drei verschiedenen Signalgruppen innerhalb von fünf Minuten** ausgelöst. Wiederholte Meldungen desselben Typs zählen für die Punkte, aber nicht als zusätzliche Signalgruppe. Spieler-Spawns und vom Framework genehmigte Haus-Teleports erhalten eine kurze Ausnahmezeit. Aktive Sperren werden in `bo_ac_bans` gespeichert und beim Verbinden erneut geprüft.

Webhook-Meldungen sind ausgehend und informieren über automatische Banns und Entbannungen; ein Webhook kann selbst keine Discord-Nachrichten entgegennehmen. Für `/bo-unban` gibt es den separaten Bot in `resources/[bo]/bo_ac_discord`. Er benötigt Node.js 20+, seine eigene Discord-Bot-Anwendung und einen HTTPS-Reverse-Proxy zur FiveM-API. Das Botkonto prüft eine Discord-Adminrolle oder freigegebene Benutzer-ID; der FiveM-Endpunkt verlangt zusätzlich ein gemeinsames Geheimnis. Stelle `bo_ac_api_token` ausschließlich mit `set` in `server.cfg` ein, nie mit `setr`, und veröffentliche den HTTP-Endpunkt nur über HTTPS. Bei der API nicht benötigte FiveM-Routen sollten per Reverse Proxy gesperrt werden.

Bot starten:

1. Erstelle eine Discord-Anwendung mit Bot und installiere sie mit Slash-Command-Berechtigung im Admin-Server.
2. Kopiere `.env.example` zu `.env` in `resources/[bo]/bo_ac_discord`; trage Bot-Token, Anwendungs-ID, Server-ID und mindestens eine freigegebene Admin-Rollen-ID oder Benutzer-ID ein. `.env` nicht veröffentlichen.
3. Erzeuge ein zufälliges Geheimnis mit mindestens 32 Zeichen. Setze denselben Wert in `.env` als `BO_AC_API_TOKEN` und in `server.cfg` mit `set bo_ac_api_token "..."`.
4. Trage in `BO_SERVER_API_URL` die öffentliche HTTPS-Adresse mit Pfad `/bo-ac/unban` ein. Leite ausschließlich diesen Pfad an den FiveM-Server weiter.
5. Optional: Setze `bo_ac_webhook` auf einen privaten Discord-Incoming-Webhook für Bann-/Entbannungsprotokolle.
6. Im Botordner `npm install` und anschließend `npm start`; nur ausgegebene Erfolgsmeldungen bestätigen eine Aktion.

Manuelle Entbannung ist auch über die Serverkonsole möglich: `boac unban license:<Hex-Lizenz-ID> <Grund>`. Im Spiel können ACE-Admins denselben Befehl ausführen. Discord: `/bo-unban` mit BO-Profil-ID aus der privaten Bannmeldung und Begründung. Der Bot übermittelt nur die Profil-ID; die FiveM-Lizenz bleibt auf dem Server.

Die Erkennung bleibt heuristisch und ist kein Ersatz für serverautoritative Spiellogik: Modifizierte Clients, OneSync-Replikation, Lags und andere Resources können Fehlmeldungen oder übersehene Cheats verursachen. Die automatische Sperre ist deshalb bewusst auf mehrere unabhängige Signale ausgelegt, aber **nicht frei von Fehlbanns**. Kalibriere Schwellen und Watch-Zonen auf einem Testserver und prüfe die Bannprotokolle, bevor der Server öffentlich startet. Das System kann nicht garantieren, sämtliche Cheats zu entdecken oder zu verhindern.

## Laufzeit

Eine feste Zusage von dauerhaft `0.00 ms` ist nicht möglich: Laufzeit hängt von Spielerzahl, Serverlast, FXServer, Datenbank und weiteren Resources ab. Der Kern hält Leerlauf-Schleifen auf langen Intervallen und verarbeitet UI-/Spielerereignisse ereignisgesteuert. Die tatsächliche Laufzeit ist in der FXServer-Resource-Messung unter realer Last zu prüfen.

## Grenzen dieser Version

Die Jobaufträge sind einfache standortgebundene Auszahlungen, keine vollständigen Berufssimulationen. Wasser und Brot werden im Inventar geführt, aber ein Hunger-/Durst-HUD ist noch nicht enthalten; das Erste-Hilfe-Set stellt Gesundheit wieder her. Telefon und MDT sind bewusst schlanke Basissysteme, kein Smartphone mit Apps oder vollständiges Polizeiverwaltungsprodukt. Die Häuser verwenden Beispielpositionen und einfache Interior-Teleports statt instanzierter Interiors; Verkauf, Mietsystem, Möbel, Lager und Hausalarme sind nicht enthalten. Die Blitzer sind persistente Standortmarker mit serverseitiger Geschwindigkeitsmessung; ein detailliertes Blitzerprop-Modell, Straßen-/Fahrtrichtungsprüfung und ein komplettes Polizeirevier-System sind nicht enthalten.

Die Garagen-, Immobilien-, Spawn- und Arbeitspositionen in `resources/[bo]/bo_core/config.lua` sind Beispielkoordinaten der GTA-V-Karte und müssen für das gewünschte Bochum-Setting angepasst und im Spiel geprüft werden. Der Fahrzeugbesitz und Garagenstatus werden gespeichert; individuelle Fahrzeugmods, Tankstand, Schadenszustand, Abschlepphof und Fahrzeugschlüssel sind noch nicht enthalten. Häuser speichern Eigentümer und Schlüssel, verwenden aber weiterhin die konfigurierten Beispiel-Interiors statt instanzierter Wohnungen; Verkauf, Mietsystem, Möbel, Lager und Hausalarme fehlen.

Dieses Projekt enthält weiterhin keine 200 fertigen Scripts. Charakterauswahl mit Mehrfachcharakteren, vollständiger EMS-Dispatch, Strafzettel/Haft, Crafting, NPC-Shops, Kleidung, Fraktionen und weitere Spielsysteme brauchen jeweils eigene Implementierung und Tests.
