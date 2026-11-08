# Z2M Automatische Firmwareupdates

Home-Assistant-Blueprint für automatische Firmwareupdates von
Zigbee2MQTT-Geräten über MQTT. Er erkennt die Update-Entitäten automatisch
und arbeitet die verfügbaren Updates mit einer frei einstellbaren
Parallelitätsgrenze ab.

## Funktionen

- Sofort bei Verfügbarkeit oder täglich zu einer gewählten Uhrzeit starten.
- Positive ganze Parallelitätsgrenze ohne feste Obergrenze.
- Wahlweise freie Plätze laufend auffüllen oder in Gruppen aktualisieren.
- Einzelne Update-Entitäten ausschließen.
- MQTT-Basistopic frei einstellen; voreingestellt ist `MQTT`.
- Jederzeit über **Aktionen ausführen** manuell starten.

## Zwei Arten der Abarbeitung

| Einstellung | Beispiel bei Grenze 5 und zwölf Updates |
|---|---|
| Freie Plätze laufend auffüllen | Sobald eines der ersten fünf Updates beendet ist, startet das nächste. |
| Erst die ganze Gruppe abschließen | Erst die ersten fünf beenden, dann die nächsten fünf, danach die letzten zwei. |

Die Parallelitätsgrenze begrenzt laufende Updates, nicht die Warteliste.
Zwischen den Starts liegen die konfigurierte Pause und die Startbestätigung.
Die letzte Gruppe darf kleiner als die eingestellte Grenze sein.
Eine Gruppe gilt als beendet, sobald keines ihrer Updates mehr läuft;
ein beendetes Update muss nicht erfolgreich installiert sein.

## Voraussetzungen

- Home Assistant ab Version 2024.10.0.
- Zigbee2MQTT mit eingerichteter MQTT-Integration und Home-Assistant-Discovery.
- Aktivierte, verfügbare Firmwareupdate-Entitäten der Geräte.
- Eine Zigbee2MQTT-Instanz und eine zuständige Automation für diese Instanz.

Die automatische Geräteerkennung verwendet die Zigbee2MQTT-Gerätekennungen
in der MQTT-Integration. Der Blueprint ist für Zigbee2MQTT-OTA gedacht.

## Installation

### Import über GitHub

In Home Assistant **Einstellungen → Automatisierungen & Szenen → Blueprints
→ Blueprint importieren** öffnen und diese Adresse eintragen:

```text
https://github.com/jxstNo1/z2m-auto-update/blob/main/z2m_auto_ota_v3.yaml
```

Für den URL-Import muss die Datei ohne Anmeldung erreichbar sein.
Bei einem privaten Repository die manuelle Installation verwenden.

### Manuelle Installation

1. [z2m_auto_ota_v3.yaml](z2m_auto_ota_v3.yaml) herunterladen.
2. Auf Home Assistant als
   `/config/blueprints/automation/local/z2m_auto_ota_v3.yaml` speichern.
3. Automationen neu laden und eine Automation aus dem Blueprint erstellen.

## Einstellungen

| Eingabe | Bedeutung |
|---|---|
| Update-Modus | Sofort oder täglich nach Zeitplan. |
| Geplante Startzeit | Täglicher Start im Zeitplanmodus. |
| Maximale parallele Updates | Beliebige positive ganze Zahl; Standard: 1. |
| Abarbeitung der Warteschlange | Laufend auffüllen oder erst die ganze Gruppe abschließen. |
| Pause zwischen Update-Starts | Abstand zwischen zwei Anfragen; Standard: 5 Sekunden. |
| MQTT-Basistopic | Muss exakt `mqtt.base_topic` in Zigbee2MQTT entsprechen; Standard: `MQTT`. |
| Ausgeschlossene Firmwareupdates | Geräte, für die keine automatischen Updates gestartet werden sollen. |

**Das MQTT-Basistopic einschließlich Groß-/Kleinschreibung abgleichen.**
Das Blueprint-Eingabefeld ändert die Zigbee2MQTT-Konfiguration nicht.
Mit der Voreinstellung gehen Update-Anfragen an:

```text
MQTT/bridge/request/device/ota_update/update
```

Bei einer bestehenden Automation kann ein früher gespeicherter Topic-Wert
Vorrang vor dem Blueprint-Standard haben. Den Wert dort ausdrücklich prüfen.
Beim Wechsel von einer älteren Version nur eine zuständige Automation aktiv lassen.

Die geplante Uhrzeit ist eine Startzeit, kein Ende eines Updatefensters.
Bereits laufende Updates ausgeschlossener Geräte zählen bei der Grenze mit.

## Warteliste und Neustarts

Alle beim Start verfügbaren Updates werden einmal in einer Warteliste
erfasst und im selben Durchlauf abgearbeitet, sofern kein Fehler oder
Timeout den Durchlauf beendet.

Erst währenddessen neu verfügbar gewordene Updates werden im nächsten
Durchlauf erfasst: im Sofortmodus bei der nächsten Minutenprüfung nach
Ende des aktuellen Durchlaufs, im Zeitplanmodus zum nächsten geplanten Start.

Die Warteliste liegt im Arbeitsspeicher. Bei einem Neustart, Neuladen oder
Abbruch geht sie verloren. Der nächste Durchlauf erfasst weiterhin
gemeldete Updates neu; es gibt keine dauerhaft gespeicherte Jobwarteschlange.

## Fehler und Grenzen

Die Steuerung folgt den Update-Entitäten und deren `in_progress)-Rückmeldung.
Ohne Startbestätigung innerhalb von zwei Minuten endet der Durchlauf.
Beim Warten auf Kapazität oder Abschluss gilt ein Timeout von drei Stunden
pro Wartevorgang. Der Durchlauf wartet auch auf die zuletzt gestarteten Updates.

Verschwindet der Status eines in diesem Durchlauf angefragten Geräts, bleibt
dessen Platz bis zur Statusrückkehr oder zum Abbruch belegt.
Ein Abbruch stoppt bereits laufende Firmwareupdates nicht.
Ein späterer Durchlauf kann weiterhin ausstehende Updates erneut versuchen.

Außerhalb dieser Automation gestartete Updates können gleichzeitig hinzukommen.
Eine globale Sperre für andere Update-Starter gibt es nicht.

Zigbee2MQTT empfiehlt aufgrund der Netzbelastung ein OTA-Update gleichzeitig.
Der Blueprint lässt höhere Werte ausdrücklich zu.

## Validierung

Die YAML-Datei einschließlich `!input` und YAML-Anker wurde geparst und
alle Jinja-Templates wurden kompiliert. Die YAML-Aktionsfolge wurde mit
simulierten Gerätezuständen ausgeführt:

- Zwölf Updates mit Grenzen 1, 2, 5 und 100.
- Beide Abarbeitungsarten und die letzte Teilgruppe.
- Ausschlüsse, bereits laufende Updates und eine leere Warteliste.
- Fehlende Startbestätigung, verlorener Status und ungültige Grenzwerte.

Zentrale Statusausdrücke wurden zusätzlich in Home Assistants Template-Engine
geprüft. Ein echter Firmwareupdate-Test und eine Installation des Blueprints
in einer Home-Assistant-Instanz wurden nicht durchgeführt.

## Lizenz

Dieses Projekt steht unter der [GNU General Public License v3.0](LICENSE).

## Dokumentation

- [Zigbee2MQTT OTA-Updates](https://www.zigbee2mqtt.io/guide/usage/ota_updates.html)
- [Home Assistant MQTT Update](https://www.home-assistant.io/integrations/update.mqtt/)
