# Immich_Server
#### useful sites
- https://docs.immich.app/install/docker-compose/
- 

# 1. Install Ubuntu on a USB stick using rufus e.g. `ubuntu-24.04.3-live-server-amd64` image
- Partitionsschema `GPT`
## 1.1 boot PC from stick, install, setup wifi etc
- in case of crash during installation `removing previous storage devices` see [removing previous storage devices](#removing-previous-storage-devices)  
- 
## 1.2 first boot `sudo apt update && sudo apt upgrade -y`

# 2. Immich install
- https://docs.immich.app/install/docker-compose/
- Docker-Netzwerk reparieren (DNS & MTU). Ohne diesen Schritt konnte Docker die offiziellen Images nicht von GitHub herunterladen.
## Alternative Anleitung hier
`https://www.hostmycode.com/tutorials/set-up-immich-on-ubuntu-2404-with-docker`  
- Zu Punkt 2. Step 2: Install Docker and Docker Compose
hier fehlen die Absätz. das ganze sind einzelne befehlen und zwar so:

`sudo apt-get updatesudo apt-get install ca-certificates curl`  
`sudo install -m 0755 -d /etc/apt/keyrings`  
`sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`  
`sudo chmod a+r /etc/apt/keyrings/docker.asc`  
### Add the repository to Apt sources:
`echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`  
`sudo apt-get update`  
``  
``  
``  
``  
``  

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

# 3. Einrichtung Timemachine

# Ordner auf der zweiten Platte erstellen
`sudo mkdir -p /mnt/backup/timemachine`

# Rechte vergeben, damit dein User darauf zugreifen kann
`sudo chown -R 1000:1000 /mnt/backup/timemachine`
`sudo chmod -R 777 /mnt/backup/timemachine`

1. Vorbereitung auf der Haupt-HDD (sda)
Wir erstellen den Ordner für Time Machine direkt neben deinem funktionierenden Immich-Ordner.

Bash
`sudo mkdir -p /mnt/data/timemachine`
`sudo chown -R 1000:1000 /mnt/data/timemachine`
`sudo chmod -R 777 /mnt/data/timemachine`

`cd ~/timemachine-app`

1. Vorbereitung auf der Haupt-HDD (sda)
Wir erstellen den Ordner für Time Machine direkt neben deinem funktionierenden Immich-Ordner.

Bash
`sudo mkdir -p /mnt/data/timemachine`
`sudo chown -R 1000:1000 /mnt/data/timemachine`
`sudo chmod -R 777 /mnt/data/timemachine`

2. Time Machine Docker (Verifiziertes Image)
vIch habe dieses Image (mbentley/timemachine) geprüft. Es ist aktuell und öffentlich verfügbar.

#### Bash
`cd ~/timemachine-app`
Die Datei mit diesem EXAKTEN Inhalt überschreiben:

### Bash
`nano docker-compose.yml`

Kopiere diesen Block (Ich habe den Image-Namen korrigiert):

### YAML
```
services:
  timemachine:
    image: mbentley/timemachine:latest
    container_name: timemachine
    restart: always
    network_mode: host
    environment:
      - CUSTOM_USER=andi
      - CUSTOM_PASSWORD=deinpasswort  # Ändere das Passwort hier!
      - SHARE_NAME=TimeMachine
      - VOLUME_SIZE_LIMIT=1T          # Reserviert 1TB auf sda für den Mac
    volumes:
      - /mnt/data/timemachine:/opt/timemachine
```
(Speichern: Strg+O, Enter. Schließen: Strg+X).

Starten:

Bash
`docker compose up -d`

### 3. Tägliches Backup von Platte 1 (sda) auf Platte 2 (sdb)
Wir nutzen rsync, um jede Nacht eine 1:1 Kopie deiner gesamten Arbeitsplatte zu erstellen.
Manueller Test (Einmalig ausführen): Dieser Befehl spiegelt alles von deiner Hauptplatte auf die Backup-Platte.

Bash
`sudo rsync -av --delete /mnt/data/ /mnt/backup/`
Automatisierung (Cronjob): Wir lassen das System die Arbeit machen, während du schläfst.

Bash
`sudo crontab -e`
(Wähle 1 für Nano). Füge diese Zeile ganz unten ein:

Plaintext
`0 3 * * * rsync -av --delete /mnt/data/ /mnt/backup/ > /var/log/backup_sync.log 2>&1`
(Das kopiert jeden Morgen um 03:00 Uhr alle neuen Fotos und Time-Machine-Daten auf die zweite HDD).



## removing previous storage devices
- error occurs of install process has no access to drive (ssd). follow these steps
- Terminal öffnen: Drücke im Installationsbildschirm Alt + F2, um in die Konsole zu wechseln
- SSD identifizieren: Führe den Befehl lsblk aus, um die Laufwerksbezeichnung (z. B. /dev/sda) zu ermitteln
- Partitionen löschen: Führe nacheinander folgende Befehle aus (ersetze /dev/sda bei Bedarf): `sudo wipefs -a /dev/sda`
- sollte es hierbei Probleme geben machen wir folgendes
  - Eingehängte Partitionen trennen: `sudo umount -l /dev/sda*`
  - Swap-Speicher deaktivieren: `sudo swapoff -a`
  - Device-Mapper und LVM auflösen: `sudo dmsetup remove_all`
  - Löschvorgang erzwingen: `sudo wipefs -af /dev/sda ` 
- `sudo sgdisk --zap-all /dev/sda`
- `sudo dd if=/dev/zero of=/dev/sda bs=1M count=100`
- Zurückkehren: Drücke Alt + F1, um zurück zum Installer zu wechseln, und starte den Vorgang neu
- Alternativ mit live Ubuntu stick und `gparted`o.ä. die partition löschen
