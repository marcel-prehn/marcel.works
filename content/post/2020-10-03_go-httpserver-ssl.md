---
title: "Go http.Server und SSL"
date: 2020-10-03
tags: ["go", "golang", "http", "ssl"]
author: "marcel"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Wie man LetsEncrypt in Go's http.Server benutzt"
disableHLJS: true
disableShare: false
searchHidden: true
cover:
    image: "https://images.unsplash.com/photo-1560574188-6a6774965120?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=2100&q=80"
    alt: Cover Image
    relative: false
    hidden: false
---

In meinem letzten Beitrag habe ich berichtet, wie sich ein React-Frontend ins Go-Backend einbinden und gemeinsam ausliefern lässt. 
Dort war die Bereitstellung noch nicht SSL-verschlüsselt. Das übernahm bisher ein vorgelagerter NGINX als Reverse Proxy. 
Dort habe ich ja versprochen, die Verbindung im Backend schon zu verschlüsseln. Also los..

### Die Zertifikate

Ich betreibe meine Anwendung auf einem kleinen VPS, keine Cloud, nichts besonderes. 
Um also die Verbindung absichern zu können, benutze ich LetsEncrypt und den certbot. 
Dabei halte ich mich strikt an die Anweisungen von deren Website: https://certbot.eff.org/

### Mounten der Zertifikate

Sind die Zertifikate erstmal installiert, werden sie einfach in den Container gemountet. 
Ich benutze docker-compose für die Orchestrierung meiner Container. Die entsprechende Config sieht so aus:

```yaml
version: "3"
services:
  backend:
    image: image:latest
    container_name: name
    volumes:
      - /etc/letsencrypt/live/DOMAIN/fullchain.pem:/etc/letsencrypt/live/DOMAIN/fullchain.pem
      - /etc/letsencrypt/live/DOMAIN/privkey.pem:/etc/letsencrypt/live/DOMAIN/privkey.pem
    ports:
      - 443:8000
```

Cert und Key liegen per default in /etc/letsencrypt/live/DOMAIN, wobei DOMAIN natürlich eure Domain darstellt — klar! `
Ich mounte sie der Einfachheit halber an die selbe Stelle im Container.

### Nutzen der Zertifikate

Haben wir erstmal alles im Container, müssen wir nur noch dem HTTP-Server beibringen, wo er die Zertifkate für die 
Absicherung der Verbindung findet. Ich erzeuge mir also einen http.Server und konfiguriere ihn initial. 
Wenn ich ihn dann allerdings starte, benutze ich nicht wie bisher die Methode ListenAndServe(), 
sondern ListenAndServeTls(). Easy peasy!

```go
httpServer := http.Server{
   Addr:    ":8000",
   Handler: router,
}
// der alte Weg:
httpServer.ListenAndServe()

//Und jetzt mit Zertifikaten im Bauch:
cert := "/etc/letsencrypt/live/DOMAIN/fullchain.pem"
key := "/etc/letsencrypt/live/DOMAIN/privkey.pem"
httpServer.ListenAndServeTLS(cert, key)
```

Und schon ist die Verbindung sicher.

Ich mache übrigens beides, je nach Stage: über eine Umgebungsvariable gesteuert starte ich lokal ohne TLS, 
in der Produktion allerdings mit. Auch das lässt sich einfach über docker-compose steuern.

So far!