---
title: "Go Container minimalisieren"
date: 2019-04-21
tags: ["Goland", "Docker", "CI", "CD", "Pipeline", "DevOps"]
author: "marcel"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
disableHLJS: true
disableShare: false
searchHidden: true
cover:
    image: "https://images.unsplash.com/photo-1568010983241-52ce16ab2cf4?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=80"
    alt: Cover Image
    relative: false
    hidden: false
---

Was ich an Go so charmant finde? 
Man kann kleine, wirklich kleine Mircoservices bauen ohne einen Overhead zu erzeugen, wie im Java/Spring Kontext.
Daher wäre es natürlich schon, solch einen wirklich kleinen Microservice auch in Container-Form klein zu halten. 
Kleines Deployment und so.
Daher möche ich es natürlich vermeiden, ein komplettes Betriebssystem inklusive der Go Lib zu laden.

Als erstes sollte man die Go-Anwendung ohne statische Links zu den Go Libs kompilieren. 
Die Libs bauen wir einfach ins Kompilat mit ein.

```shell
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
```

Nun noch das Dockerfile. 
Ausgangspunkt ist das scratch Image, dass nur als Hülle dient und 0 Byte groß ist. 
Dann das Übliche Prozedere: Verzeichnis hinzufügen, Port freigeben, Entrypoint setzen.

```shell
FROM scratch
ADD main /
EXPOSE 8080
CMD ["/main"]]
```

Soweit so gut. 
Das entstandene Image sollte — natürlich je nach Größe der Anwendung — einige wenige Megabyte nicht überschreiten. 
Kein Vergleich zum 500MB Spring-JVM-Monster.

FYI: Falls man die Endpunkte des Microservices über HTTPS zur Verfügung stellen möchte, 
benötigt man natürlich die passenden Zertifikate. 
Wenn man diese nicht in den Container einbacken möchte, muss man ein entsprechendes Base-Image verwenden. 
Das hier benutzte scratch-Image hat diesbezüglich nichts an Board.

Es gibt aber Abhilfe: [https://github.com/drone/ca-certs](https://github.com/drone/ca-certs)

Dieses Basis-Image hat alle Zertifikate enthalten und kann als minimale Grundlage genutzt werden. Ciao!