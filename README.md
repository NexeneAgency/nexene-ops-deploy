# Nexene Ops – Deployment

Die aktuelle Deployment-Konfiguration liegt jetzt direkt im App-Repository:

- [Hostinger-Anleitung](https://github.com/NexeneAgency/nexene-ops-app/blob/main/docs/HOSTINGER.md)
- [Compose-URL für „Aus URL erstellen“](https://raw.githubusercontent.com/NexeneAgency/nexene-ops-app/main/compose.yaml)
- [Beispielkonfiguration](https://raw.githubusercontent.com/NexeneAgency/nexene-ops-app/main/deploy/env.example)

Voraussetzung: Das App-Repository ist öffentlich. Docker baut das Image direkt aus dem Git-Quellcode auf dem VPS. Für diesen Weg ist kein Registry-Login erforderlich; das vorhandene GHCR-Paket bleibt privat.

Die hier vorhandene `compose.yaml` entspricht der Hauptkonfiguration und bleibt für bisherige Links verfügbar. Für neue Installationen die URL aus dem App-Repository verwenden. Zugangsdaten liegen ausschließlich in `/opt/nexene-ops/.env` auf dem VPS.

Ein späterer Wechsel des App-Repositorys auf privat verhindert weitere anonyme URL-Importe und Git-Builds. Bereits gebaute Container laufen weiter. Ein öffentliches GHCR-Paket lässt sich laut GitHub nicht wieder privat stellen; es wird für diesen Weg deshalb nicht benötigt.
