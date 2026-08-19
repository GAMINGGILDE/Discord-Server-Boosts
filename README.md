# Discord-Server-Boosts

Ein Workflow zum Senden monatlicher automatischer Erinnerungen zum Boosten unseres Discord Servers.

## 1. Ziel

**Discord-Server-Boosts** veröffentlicht einmal pro Monat automatisch eine Nachricht im Discord Ankündigungen-Kanal.

Die Nachricht wird als Markdown-Datei (`boost-reminder.md`) innerhalb des GitHub-Repositories gepflegt. Eine **GitHub Action** wird über einen Zeitplan automatisch ausgeführt, liest die Nachricht aus der Markdown-Datei und sendet sie über einen **Discord-Webhook** an den gewünschten Discord-Kanal.
Der aktuelle Monat wird beim Versand automatisch ermittelt und kann innerhalb der Nachricht über den Platzhalter `{{MONTH}}` verwendet werden.

Der Ablauf sieht vereinfacht so aus:

```text
GitHub Repository
      │
      ├── boost-reminder.md
      │
      ▼
GitHub Actions
      │
      ├── aktuellen Monat ermitteln
      ├── {{MONTH}} ersetzen
      └── Nachricht aufbereiten
      │
      ▼
Discord Webhook
      │
      ▼
Discord-Kanal
```

---

## 2. Voraussetzungen

Für die Einrichtung werden benötigt:

* Discord-Server
* Berechtigung zum Erstellen bzw. Verwalten von Webhooks
* GitHub-Repository
* Berechtigung zum Anlegen von GitHub Actions und Repository Secrets

---

# 3. Einrichtung in Discord

## 3.1 Webhook erstellen

Zunächst wird im gewünschten Discord-Kanal ein Webhook eingerichtet.
Dazu die Einstellungen des entsprechenden Kanals öffnen und zu den **Integrationen** wechseln.

Dort:

1. **Webhooks** auswählen.
2. Einen neuen Webhook erstellen.
3. Einen Namen vergeben, beispielsweise `GAMING GILDE` (aktuell).
4. Den gewünschten Discord-Kanal auswählen.
5. Optional ein Profilbild für den Webhook festlegen (Gaming Gilde Logo).
6. Die **Webhook-URL kopieren**.

Die Webhook-URL ermöglicht das Senden von Nachrichten an den entsprechenden Kanal.

> **Wichtig:** Die Webhook-URL ist wie ein Zugangsschlüssel zu behandeln und darf nicht öffentlich veröffentlicht oder direkt im GitHub-Repository gespeichert werden.

---

# 4. Einrichtung in GitHub

## 4.1 Repository Secret anlegen

Die Discord-Webhook-URL wird als verschlüsseltes Repository Secret hinterlegt.
Im entsprechenden GitHub-Repository:

**Settings → Secrets and variables → Actions**

Unter **Repository secrets** wird anschließend ein neues Secret angelegt.

Name:

```text
DISCORD_WEBHOOK_URL
```

Als Wert wird die zuvor aus Discord kopierte Webhook-URL eingetragen.
Die GitHub Action kann anschließend über

```yaml
${{ secrets.DISCORD_WEBHOOK_URL }}
```

auf die URL zugreifen, ohne dass diese im Quellcode des Repositorys sichtbar ist.

---

# 5. Markdown-Datei für die Nachricht

Die eigentliche Discord-Nachricht befindet sich in der Datei:

```text
boost-reminder.md
```

Die Datei wird im Hauptverzeichnis des Repositorys abgelegt.

Beispiel:

```text
repository/
│
├── boost-reminder.md
│
└── .github/
    └── workflows/
        └── boost-reminder.yml
```

Der Inhalt der Nachricht kann damit unabhängig vom Workflow bearbeitet werden.

Beispiel:

```markdown
# 🚀 Community-Boost im {{MONTH}}

Auch im **{{MONTH}}** möchten wir unsere Community gemeinsam weiter wachsen lassen!

Mit einem Server-Boost hilfst du dabei, unseren Discord-Server weiter wachsen zu lassen und Vorteile für die gesamte Community freizuschalten.

Vielen Dank für eure Unterstützung! 🤩
```

## 5.1 Platzhalter für den aktuellen Monat

Innerhalb der Markdown-Datei kann an beliebigen Stellen

```text
{{MONTH}}
```

verwendet werden.

Die GitHub Action ersetzt diesen Platzhalter unmittelbar vor dem Versand durch den aktuellen deutschen Monatsnamen.

Beispielsweise wird aus:

```text
Community-Boost im {{MONTH}}
```

im August:

```text
Community-Boost im August
```

Dadurch muss die Markdown-Datei beim Monatswechsel nicht manuell angepasst werden.

---

# 6. GitHub Action

Die GitHub Action befindet sich unter:

```text
.github/workflows/boost-reminder.yml
```

Der Workflow übernimmt folgende Aufgaben:

1. Automatischer Start zum festgelegten Zeitpunkt.
2. Checkout des GitHub-Repositories.
3. Ermittlung des aktuellen Monats.
4. Umwandlung der Monatsnummer in den deutschen Monatsnamen.
5. Ersetzung von `{{MONTH}}` innerhalb von `boost-reminder.md`.
6. Umwandlung des Nachrichtentextes in ein JSON-Payload.
7. Versand des Payloads an den Discord-Webhook.

Eine manuelle Ausführung über GitHub Actions ist zusätzlich möglich und dient insbesondere zum Testen der Implementierung.

---

# 7. Zeitsteuerung

Die automatische Ausführung wird durch folgende Cron-Konfiguration festgelegt:

```yaml
schedule:
  - cron: "0 18 1 * *"
```

Diese Konfiguration bedeutet:

```text
Minute:      0
Stunde:      18
Tag:         1
Monat:       jeder
Wochentag:   beliebig
```

Der Workflow wird damit **am 1. jedes Monats um 18:00 Uhr UTC** ausgeführt.

Zu beachten ist, dass der Zeitplan von GitHub Actions in **UTC** angegeben wird. Durch Sommer- und Winterzeit entspricht dies nicht ganzjährig derselben deutschen Ortszeit.

---

# 8. Manuelles Testen

Durch

```yaml
workflow_dispatch:
```

kann die Action zusätzlich jederzeit manuell gestartet werden.

Dazu im GitHub-Repository:

**Actions → Discord Server Boosts Erinnerung → Run workflow**

Nach dem Start kann der ausgeführte Job in GitHub geöffnet werden. Dort sind die einzelnen Arbeitsschritte und deren Status sichtbar.
Bei erfolgreicher Ausführung sollte die Nachricht wenige Sekunden später im konfigurierten Discord-Kanal erscheinen.

---

# 9. Sicherheit

Die Discord-Webhook-URL darf nicht innerhalb von

* `boost-reminder.md`,
* `boost-reminder.yml`,
* anderen Repository-Dateien,
* Commits oder
* öffentlich zugänglicher Dokumentation

hinterlegt werden.

Stattdessen wird ausschließlich das GitHub Secret

```text
DISCORD_WEBHOOK_URL
```

verwendet.

Sollte eine Webhook-URL versehentlich veröffentlicht werden, sollte der betreffende Webhook in Discord gelöscht bzw. erneuert und anschließend das GitHub Secret aktualisiert werden.

---

# 10. Pflege der monatlichen Nachricht

Für reguläre Textänderungen muss die GitHub Action nicht angepasst werden.

Es genügt, die Datei

```text
boost-reminder.md
```

zu bearbeiten und die Änderung in das Repository zu übernehmen.

Beim nächsten automatischen Lauf verwendet die GitHub Action automatisch die aktuelle Version der Datei.

Dadurch sind **Inhalt und technische Implementierung voneinander getrennt**:

```text
boost-reminder.md
        │
        └── Inhalt und Formatierung

boost-reminder.yml
        │
        └── Zeitsteuerung und Versandlogik

DISCORD_WEBHOOK_URL
        │
        └── geschützter Zugang zum Discord-Webhook
```

## Hinweis zum Discord-Zeichenlimit

Beim Versand über das Feld `content` gelten die normalen Discord-Limits für Nachrichten. Insbesondere darf eine einzelne Nachricht maximal **2.000 Zeichen** umfassen.

Soll die Boost-Erinnerung dieses Limit überschreiten, muss der Workflow erweitert werden, sodass der Inhalt auf mehrere Discord-Nachrichten aufgeteilt oder alternativ über andere Discord-Nachrichtenformate ausgegeben wird.
