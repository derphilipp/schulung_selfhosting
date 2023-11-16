---
theme: seriph
# background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: prism
lineNumbers: false
info: |
  ## Schulung Selfhosting
drawings:
  persist: false
transition: slide-left
title: Schulung Selfhosting
mdc: true

---

# Schulung Selfhosting

## Server zu Hause, statt IRGENDWO in der Cloud


---
transition: fade-out
---

# Agenda

* Überblick: Warum Selfhosting?
* Portfreigabe und Dyndns-Dienste
* Caddy als Reverse-Proxy für HTTPS
* Docker-compose Basics und Systemd-Template
* Konfigurieren und Betreiben von Diensten
* Weiter Übungen und individuelle Fragen

---
layout: default
---

# Motivation

* Unabhängigkeit von Drittanbietern
* Volle Kontrolle über die Daten
* Anpassungsfähigkeit und Flexibilität
* Kosteneffizienz
* Lernmöglichkeiten und Fähigkeitenentwicklung

---

# Warnungen

* Selfhosting ist nicht für jeden geeignet
* Selfhosting ist nicht für jeden Dienst geeignet
* Selfhosting ist nicht immer die einfachste/beste/günstigste/sicherste Lösung
* Wir schauen uns EINE Art an es zu machen

---

# Betriebssystem

* Ein beliebiges Linux-Betriebssystem
* Empfehlungen:
  * Debian Linux (stabil, einfach, viele Pakete, viele Anleitungen, viele Nutzer)
  * Raspberry Pi OS (baut auf Debian auf, für Raspberry Pi optimiert)
  * Ubuntu Server (stabil, einfach, viele Pakete)
* Prinzipiell geht auch jedes andere Linux-Betriebssystem (Sytemd und Docker müssen laufen)
* Gentoo, Arch Linux und NixOS sind tolle Betriebssysteme, aber nicht für Anfänger geeignet
* Für heutigen Kurs: Debian Linux, DVD-Image (iso) herunterladen und installieren

---

# Hardware

* In diesem Kurs verwende ich eine Virtuelle Maschine mit Debian Linux
* Es kann aber auch andere Systeme verwendet werden
* Wichtig: Systemd und Docker müssen laufen
* Wir brauchen später den Pfad von `docker-compose` (je nach System Unterschiedlich)
* 1 GB Ram sollte ausreichen für Betriebssystem und Basis-Dienste
* Bei Resourcenhungrigen Diensten (z.B. Nextcloud) mehr Ram verwenden
* Der Computer / Raspberry Pi / VM braucht eine eigene IP-Adresse im lokalen Netzwerk (statisch/fest vergeben)

---

# Hardware anschaffen - Tipps

* Raspberry Pi 4/5, aber System auf externe SSD installieren
* Gebrauchte Hardware, oft sind Siemens, HP oder Dell Büro Desktops günstig zu bekommen (ca. 100-150€)
* Alternativ für diesen Kurs kann eine Virtuelle Maschine verwendet werden (z.B. Virtualbox)
* Tipp: Bei etwas potentem Rechner: Proxmox VE installieren und VMs erstellen
  * Für verschiedene Anwendungszwecke verschiedene Betriebssysteme
* Achtung: ARM Prozessoren (z.B. Raspberry) sind oft günstig / energiesparend, aber manche Docker Images gibt es nicht dafür (inzwischen sehr selten)

---

# Portfreigabe

* Grundlagen der Portfreigabe
* Konfiguration des Routers für Selfhosting
* Auswahl und Einrichtung von Dyndns-Diensten
* Sicherheitsaspekte der Portfreigabe

---

# Konventionen

* Wir verwenden den Pfad `/opt/data` für alle Daten
* Wir verwenden den Pfad `/opt/dockerfiles` für alle docker-compose Dateien
* Wir verwenden den Namen `dc@.service` für unsere Template-Datei

---

# Portfreigabe 1

* Portfreigabe ist notwendig, um Dienste von außen erreichbar zu machen
* Manche Service Provider erlauben keinerlei Portfreigaben
* Portfreigabe ist ein Sicherheitsrisiko, sollten also vorsichtig eingesetzt werden

---

# Portfreigabe 2

* Zumeist muss ein Router so eingerichtet werden, dass dieser Anfragen an einen bestimmten Port an einen bestimmten Rechner im lokalen Netzwerk weiterleitet
* Die IP-Adresse des Rechners sollte statisch sein, damit die Portfreigabe nicht bei jeder Änderung der IP-Adresse neu eingerichtet werden muss
  * Entweder immer die gleiche IP adresse vergeben (z.B. via Router)
  * Oder IP Adresse statisch vergeben
* Die Portfreigabe sollte nur für die Ports eingerichtet werden, die tatsächlich benötigt werden
* Die Portfreigabe sollte nur für die Protokolle eingerichtet werden, die tatsächlich benötigt werden

---

# Portfreigabe 3

* Falls kein Vertrag mit statischer IP (teuer!) vorhanden ist, muss eine dynamische IP-Adresse mit einem Dyndns-Dienst verknüpft werden
* Dyndns Dienst verknüpft eine Domain mit einer IP-Adresse
* Der Router muss die IP-Adresse an den Dyndns-Dienst melden
* Der Router muss die IP-Adresse regelmäßig an den Dyndns-Dienst melden
* Beispiel für Dyndns Dienst: duckdns.org

---

# Problematik Portfreigabe

* Nun muss für jeden einzelnen Dienst eine Portfreigabe eingerichtet werden
* Zudem wird jeder Dienst via http ausgeliefert
  * Unverschlüsselt
  * Unsicher
  * Viele Dienste verweitern den Dienst / müssen jedes Mal "unsicherheit akzeptieren" etc.
* Lösung: Reverse Proxy

---

# DNS Konfiguration

* DNS ist das System, das Domainnamen in IP-Adressen auflöst
* Wir brauchen eine Zuordnung von einer Domain auf unsere IP, damit der Reverse Proxy die Anfragen an die richtigen Dienste weiterleiten kann
* Problem: Wir haben keine statische IP-Adresse
* Aber: Wir haben einen fixen Domainnamen durch den Dyndns-Dienst
* Lösung: Wir verwenden einen CNAME-Eintrag
* Wir setzen ALLE unsere Dienste mit EIGENEN Domains auf den CNAME-Eintrag des Dyndns Dienstes
* Beispiel:
  * `audiobookshelf.meinewunderbaredomain.de CNAME meineigenerusername.duckdns.org`
  * `nextcloud.meinewunderbaredomain.de CNAME meineigenerusername.duckdns.org`

---

# Reverse Proxy 1

* Ein Reverse Proxy ist ein Server, der Anfragen an andere Server weiterleitet
* Der Reverse Proxy kann Anfragen anhand der Domain auf verschiedene Server weiterleiten
* Reverse Proxies z.B. nginx, traefik, caddy
* Hier im Kurs verwenden wir caddy

---

# Reverse Proxy 2

* Vorteile Caddy:
  * Einfache Konfiguration
  * Automatische HTTPS-Konfiguration
  * Automatische Zertifikatsverlängerung
  * Open Source
  * Resourcenschonend

---


# Beispiel Setup

```mermaid
graph LR
    A[Internet] -->|Ports 80,443| B[DSL Router]
    B --> |Port Forwarding| D[Caddy]
    D --> E[Nextcloud Container]
    D --> F[Audiobookshelf Container]
    D --> G[Weitere Container]

    subgraph Homeserver mit Docker
    D
    E
    F
    G
    end

```
---

# Caddy

* Einführung in Caddy
* Automatische HTTPS-Konfiguration
* Einrichtung als Reverse-Proxy

---

# Docker Allgemein 1

* Containerdienst
* Keine Virtualisierung des vollen Betriebssystems
* Nicht komplett neu, aber sehr populär geworden
* Einfache Verwaltung von Containern
* Extrem viele Dienste als Container verfügbar
* Alternativen: Podman, LXC, LXD
  * Etliche davon nutzen identische Images, sind also voll kompatibel!
* Lokale Sicherheit:
  * Achtung bei der Verwendung von Docker: Haben lokale Benutzer Rechte auf Docker können sie root-Rechte erlangen!
  * Images aus fremden Quellen können Schadcode enthalten

---

# Docker Allgemein 2

* Image: Ein Image ist eine Vorlage für einen Container (z.B. ein Webserver Image)
* Container: Ein Container ist eine isolierte Umgebung, die eine Anwendung ausführen kann
* Vergleich bei Programmiersprachen: Image ist wie eine Klasse, Container ist wie ein Objekt
* `Dockerfile`: Eine Datei, die ein Image beschreibt (wenn man z.B. selbst ein Image erstellen möchte)

---

# Docker Allgemein 3

* `docker-compose`: Tool um mehrere Container zu verwalten
* `docker-compose.yml`: Eine Datei im YAML Format, die mehrere Container beschreibt
* Volumes
  * Ein **Ordner**, der von einem Container verwendet wird
  * Kann in einen **lokalen Ordner** gemountet werden
  * Kann auch als **Volume-Container** verwendet werden
  * Es können auch **Geräte** und **Dienste** gemountet werden (z.B. X-Server, Drucker, ...)

---

# Docker Allgemein 4

* Fertige Docker Images können aus dem Internet heruntergeladen werden
* Manchmal gibt es auch Dockerfile Dateien, mit denen man selbst Images erstellen kann
* Oft gibt es fertige Images für Dienste
* Diese finden sich oft auf [Docker Hub](https://hub.docker.com/)

---

# Docker Allgemein 5

* Tags
  * Images haben neben ihrem Namen noch einen Tag (z.B. `nginx:latest` oder `nginx:1.19.10`)
  * Immer `latest` zu nehmen ist nicht immer empfehlenswert, da sich die Images ändern können. Updatepfade beachten!
  * Niemals zu aktualisieren ist nicht empfehlenswert, da Sicherheitslücken auftreten können
  * ➡️ Kompromiss aus aktualisieren und Komfort finden
  * Varianten
    * Oft gibt es verschiedene Varianten eines Images (z.B. `nginx:alpine`, `nginx:1.19.10-alpine`)
    * Diese Beschreiben oft das Basis-Image auf dem die Software Aufbaut (oft: `alpine` oder `debian`)
    * Macnhmal auch die verwendete Technolgie (z.B. `nextcloud:fpm` oder `nextcloud:apache`)

---

# Docker-compose allgemein

* Einfache Verwaltung von mehreren Containern
  * Beispiel: Nextcloud benötigt Datenbank, Redis, Cronjob, ...
  * Konfiguration aller für einen Dienst benötigten Container in einer Datei
  * Einfaches Starten und Stoppen aller Container
* Oft gibt es fertige docker-compose Dateien für Dienste
* Wir verwenden in diesem Kurs fast ausschließlich fertige docker-compose Dateien
* Weitere docker-compose Dateien finden wir oft bei den jeweiligen Projekten (z.B. Nextcloud)

---

# Systemd allgemein

* Systemd ist ein Dienst, der auf vielen Linux-Systemen läuft
* Systemd ist für das Starten und Stoppen von Diensten zuständig
* Systemd kann auch Container starten und stoppen
* Mit Systemd lassen sich einfache Skripte für das Starten und Stoppen von Containern schreiben
* Wir schreiben ein Systemd-Template für das Starten und Stoppen von Containern

---

# Docker installieren

* Befehler als root ausführen oder mit `sudo` voranstellen
* `apt update`
* `apt install nano` (für einen einfachen Editor)
* `apt install docker docker-compose` (für Docker und Docker-compose)
* Testen: `docker pull alpine` (ggf. mit `sudo`)

---

# Docker ausführen

Beispielcontainer:

```sh
docker run --rm -p 1234:80 tutum/hello-world
```

* `--rm` entfernt den Container nach dem Beenden
* `-p 1234:80` leitet den Port 1234 auf Port 80 im Container weiter
* `tutum/hello-world` ist der Name des Images, das wir verwenden
* Nun können wir auf [http://IP_ADRESSE:1234](http://IP_ADRESSE:1234) zugreifen

```mermaid
graph LR
    A[Netzwerk] -->|Port 1234| B[Netzwerk-Interface]
    B --> |Port 80| C[Container]
    subgraph Host
    B
    C

    end

```

---

# Docker-compose Einführung

* Wenn wir den gleichen Dienst nun mit `docker-compose` nutzen wollen, brauchen wir eine `docker-compose.yml` Datei

```yaml
version: "3.7" # Versionsnummer von docker-compose

services:
  hello-world: # Beliebig wählbarer Name unserers Dienstes
    image: tutum/hello-world # Name des Images
    ports:
      - "1234:80" # Portweiterleitung
```

* Befehl zum Starten:

```sh
docker-compose up
```

---

# Docker Helfer

⚠️ Bei allen Helfern muss Zugriff auf Docker erfolgen. Dies wird oft mit dem mounten des Docker-Sockets erreicht. Dies ist ein Sicherheitsrisiko - daher dürfen die Tools niemals ins Internet freigegeben werden.

* [Lazydocker](https://github.com/jesseduffield/lazydocker): Terminal UI für Docker
* [Portainer CE](https://www.portainer.io/) (Community Edition): Web UI für Docker


---

# Portainer als Beispiel

`docker-compose.yml`:

```yaml
version: "3"
services:
  portainer:
    image: portainer/portainer-ce:latest
    ports:
      - 9443:9443
      volumes:
        - /opt/data/portainer/data:/data
        - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```

* Volumes binden den Ordner `/opt/data/portainer/data` auf den Ordner `/data` im Container. Dort speichert Portainer seine Daten
* Der Ordner `/var/run/docker.sock` ist der Socket, über den Portainer mit Docker kommuniziert -> Wir geben diesen Ordner in den Container

---

# Systemd Template

```systemd
[Unit]
Description=%i service with docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always
TimeoutStartSec=1200

WorkingDirectory=/opt/dockerfiles/%i

# Remove old containers, images and volumes and update it
ExecStartPre=/usr/bin/docker-compose down -v
ExecStartPre=/usr/bin/docker-compose rm -fv
ExecStartPre=/usr/bin/docker-compose pull

# Compose up
ExecStart=/usr/bin/docker-compose up

# Compose down, remove containers and volumes
ExecStop=/usr/bin/docker-compose down -v

[Install]
WantedBy=multi-user.target
```

---

# Systemd template verweden

* Kopieren des Templates in eine neue Datei unter dem Pfad `/etc/systemd/system/dc@.service`
* Um einen konkreten Dienst anzulegen muss eine Datei unter `/opt/dockerfiles/<name>/docker-compose.yml` angelegt werden
* Um den Dienst nun zu verwenden muss `systemctl enable dc@<name>` ausgeführt werden. Damit ist der Dienst automatisch beim Starten des Systems aktiviert
* Mit `systemctl disable dc@<name>` können wir den Dienst deaktivieren
* Ab dann können wir mit `systemctl start dc@<name>` und `systemctl stop dc@<name>` starten und stoppen
* Mit `systemctl restart dc@<name>` können wir den Dienst neu starten
* Mit `systemctl status dc@<name>` können wir den Status des Dienstes abfragen

---

# Docker-compose

* Grundlagen von `docker-compose`
* Erstellen und Verwalten von Docker-Containern
* Integration mit Systemd für automatisches Management
* Best Practices und Tipps

---

# Dienste

* Auswahl geeigneter Dienste für Selfhosting
* Grundlegende Konfigurationstechniken
* Überwachung und Wartung von Diensten
* Troubleshooting und Problembehebung

---


# Diagramme

```plantuml {scale: 0.7}
@startuml

package "Some Group" {
  HTTP - [First Component]
  [Another Component]
}

node "Other Groups" {
  FTP - [Second Component]
  [First Component] --> FTP
}

cloud {
  [Example 1]
}


database "MySql" {
  folder "This is my folder" {
    [Folder 3]
  }
  frame "Foo" {
    [Frame 4]
  }
}


[Another Component] --> [Example 1]
[Example 1] --> [Folder 3]
[Folder 3] --> [Frame 4]

@enduml
```

---

# Caddy docker-compose

```yaml
version: "3.7"

services:
  caddy:
    image: caddy:latest
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - /opt/data/caddy/Caddyfile:/etc/caddy/Caddyfile
      - /opt/data/caddy/logs:/srv/log
      - /opt/data/caddy/data:/data
      - /opt/data/caddy/caddy_config:/config
```

---
src: ./pages/multiple-entries.md
hide: false
---

---
layout: center
class: text-center
---

# Learn More

[Documentations](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/showcases.html)
