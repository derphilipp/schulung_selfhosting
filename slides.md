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

# Portfreigabe

* Grundlagen der Portfreigabe
* Konfiguration des Routers für Selfhosting
* Auswahl und Einrichtung von Dyndns-Diensten
* Sicherheitsaspekte der Portfreigabe

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
    A[Internet] -->|Port 443| B[DSL Router]
    B --> |Port Forwarding| D[Caddy]
    D --> E[Nextcloud Container]
    D --> F[Audiobookshelf Container]

    subgraph Homeserver mit Docker
    D
    E
    F
    end

```
---

# Caddy

* Einführung in Caddy
* Automatische HTTPS-Konfiguration
* Einrichtung als Reverse-Proxy

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

[Learn More](https://sli.dev/guide/syntax.html#diagrams)

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
