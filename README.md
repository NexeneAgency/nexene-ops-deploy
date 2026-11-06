# Nexene Ops – Hostinger Deployment

Öffentliche Deployment-Konfiguration. Der Anwendungscode und das Container-Image sind privat. Diese Dateien enthalten keine Zugangsdaten. Voraussetzung ist ein Linux/amd64-VPS mit Docker und einer HTTPS-Domain über einen vorhandenen Reverse Proxy.

## 1. Privates Image freigeben

Einen GitHub Personal Access Token (classic) mit ausschließlich `read:packages` für einen Benutzer erstellen, der das private Container-Paket lesen darf. Keine `repo`- oder Schreibberechtigungen für den VPS vergeben. Falls GitHub für eure Organisation SSO verlangt, den Token dafür freigeben.

In Hostingers Webkonsole als root (oder zuerst `sudo -i`):

```sh
docker login ghcr.io -u DEIN_GITHUB_BENUTZERNAME
```

Bei `Password` den Token eingeben. Nicht in einen Befehl mit sichtbarem Token schreiben und nicht hier oder im Chat teilen.

Danach prüfen:

```sh
docker pull ghcr.io/nexeneagency/nexene-ops-app:latest
```

Falls die Anmeldung für einen anderen Linux-Benutzer erfolgt, muss Hostingers Docker Manager dessen Registry-Zugang verwenden können. Maßgeblich ist, ob der Docker Manager das private Image tatsächlich abrufen kann; ein erfolgreicher Pull in einer anderen Benutzerkonfiguration allein garantiert das nicht.

## 2. Zugangsdaten einmalig auf dem VPS hinterlegen

In derselben Root-Webkonsole:

```sh
install -d -m 700 /opt/nexene-ops
curl --fail --show-error --location \
  https://raw.githubusercontent.com/NexeneAgency/nexene-ops-deploy/main/env.example \
  -o /opt/nexene-ops/.env
chmod 600 /opt/nexene-ops/.env
nano /opt/nexene-ops/.env
```

Alle Beispielwerte ersetzen: die tatsächliche HTTPS-Adresse, die drei erlaubten Admin-E-Mail-Adressen, SMTP-Host und SMTP-Zugangsdaten sowie den erlaubten Absender. Mehrere Admin-Adressen mit Komma trennen. Port 587 verwendet STARTTLS und `SMTP_SECURE=false`; für Port 465 `SMTP_SECURE=true`.

Die Datei gehört auf den VPS, nicht in dieses öffentliche Repository. Ohne sie kann der URL-Import die App nicht starten. Keine echte Konfiguration in Hostingers URL-Feld eintragen.

## 3. In Hostinger aus URL erstellen

**URL:**

```text
https://raw.githubusercontent.com/NexeneAgency/nexene-ops-deploy/main/compose.yaml
```

**Anwendungsname:** `nexene-ops`

Dann **Bereitstellen** wählen. Das Image muss zuvor vom privaten App-Repository erfolgreich veröffentlicht worden sein. Wenn Hostinger bei der Registry `unauthorized` meldet, die Registry-Anmeldung des Managers prüfen; dafür muss das Image nicht öffentlich gemacht werden.

## 4. Domain und HTTPS

Die Compose-Datei öffnet die App nur auf `127.0.0.1:3000`. DNS und der Reverse Proxy müssen eure HTTPS-Domain darauf weiterleiten. Ein Beispiel für einen bereits auf dem Host installierten Caddy:

```caddy
ops.example.com {
    reverse_proxy 127.0.0.1:3000
}
```

Die Beispiel-Domain ersetzen und identisch in `APP_URL` eintragen. Ein in Docker laufender Proxy benötigt stattdessen ein gemeinsames Docker-Netzwerk; dessen `127.0.0.1` ist ein anderer Netzwerkraum. Diese Vorlage verändert keine vorhandenen Websites und startet keinen eigenen Proxy.

## 5. Prüfen und sichern

```sh
curl --fail http://127.0.0.1:3000/api/health
```

Erwartet: `{"ok":true}`. Danach eure HTTPS-Adresse öffnen und mit einer hinterlegten Admin-Adresse den Anmeldelink anfordern.

In Hostingers Container-Konsole kann eine konsistente Sicherung erzeugt werden:

```sh
node scripts/backup.mjs
```

Sie landet im Backup-Volume. Regelmäßige, externe Sicherungen zusätzlich einrichten; ein Volume auf demselben VPS schützt nicht vor dem Verlust des VPS. Wiederherstellung zunächst separat prüfen.

## Updates

Nach einem erfolgreichen Build im privaten App-Repository in Hostinger erneut bereitstellen und das neue Image abrufen. Die Datenvolumes behalten. Für einen gezielten Rollback statt `latest` ein bekanntes `sha-<Commit-SHA>`-Tag verwenden. Vor Updates sichern und Kompatibilität der Datenbankmigrationen beachten. Ein Code-Rollback allein macht eine Datenbankmigration nicht rückgängig.

Die Volumes heißen fest `nexene_ops_data` und `nexene_ops_backups`. Nur eine Instanz mit diesen Namen betreiben. Beim Löschen der Anwendung die Volumes nicht entfernen, wenn die Daten erhalten bleiben sollen.
