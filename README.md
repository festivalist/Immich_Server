# Immich_Server
Immich Server für Andi

# 1. Install Ubuntu on a USB stick using rufus e.g. `ubuntu-24.04.3-live-server-amd64` image
## 1.1 boot PC from stick, install, setup wifi etc
## 1.2 first boot `sudo apt update && sudo apt upgrade -y`

# 2. Immich install
Docker-Netzwerk reparieren (DNS & MTU). Ohne diesen Schritt konnte Docker die offiziellen Images nicht von GitHub herunterladen.

## 2.1 Konfigurationsdatei öffnen:

Bash `sudo nano /etc/docker/daemon.json`

Diesen Inhalt einfügen (behebt DNS-Blockaden und Paketverlust):

```JSON
{
  "dns": ["8.8.8.8", "1.1.1.1"],
  "mtu": 1400
}
```
### Docker neu starten:

### Bash
`sudo systemctl restart docker`

## 2.2 Die Verzeichnisse auf der 2-TB-HDD vorbereiten
Immich verweigert den Start, wenn die Marker-Dateien und Ordner auf der Festplatte (/dev/sda) fehlen.

### Ordnerstruktur anlegen:

Bash
```
sudo mkdir -p /mnt/data/immich/{library,upload,profile,thumbs,backups,encoded-video}
```

Sicherheits-Marker (.immich) erstellen: Dies signalisiert Immich, dass der Mount-Punkt korrekt ist:

Bash

`sudo touch /mnt/data/immich/.immich`
`sudo touch /mnt/data/immich/library/.immich`
`sudo touch /mnt/data/immich/upload/.immich`
`sudo touch /mnt/data/immich/profile/.immich`
`sudo touch /mnt/data/immich/thumbs/.immich`
`sudo touch /mnt/data/immich/backups/.immich`
`sudo touch /mnt/data/immich/encoded-video/.immich`
Berechtigungen für den Docker-Nutzer (UID 1000) setzen:

Bash
`sudo chown -R 1000:1000 /mnt/data/immich`
`sudo chmod -R 775 /mnt/data/immich`

## 2.3. Installation der Immich-Konfiguration
Wir haben die offizielle Docker-Compose Methode gewählt, da die CasaOS-App-Store Version veraltet war.

###Installationsordner erstellen und betreten:

Bash
`mkdir -p ~/immich-app && cd ~/immich-app`

### Die Konfigurationsdateien herunterladen:

Bash
`wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml`
`wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env`

Die .env-Datei an dein Setup anpassen:

Bash
`nano .env`
Ändere dort folgende Zeilen ab (nutze absolute Pfade):

Plaintext
`UPLOAD_LOCATION=/mnt/data/immich`
`DB_DATA_LOCATION=/DATA/AppData/immich/postgres`

(Speichern mit Strg+O, Schließen mit Strg+X).

## 2.4. Immich-Server starten
Container hochfahren:

Bash
`docker compose up -d`
Status prüfen:

Bash
`docker ps`
Alle vier Container (immich_server, immich_machine_learning, immich_postgres, immich_redis) müssen nun den Status "Up (healthy)" haben.

5. Zugriff & Abschluss
Weboberfläche: Öffne http://192.168.0.9:2283 im Browser.

Speicher-Check: In den Einstellungen unter "Administration" -> "Storage" müssen nun die 1,7 TB deiner HDD angezeigt werden.
