---
title: "Go und React gemeinsam ausliefern"
date: 2020-09-17
tags: ["Golang", "React", "Docker"]
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
    image: "/covers/gears.png"
    alt: Cover Image
    relative: false
    hidden: false
---

Für ein aktuelles Projekt entwickle ich einen Webservice bestehend aus einem Go-Backend und einer entsprechenden React-UI. 
Bisher habe ich Frontend und Backend als separate Container ausgeliefert, meist in Kombination mit einem NGINX als Reverse Proxy. 
Das war in sofern praktisch, als das die einzelnen Services unabhängig von einander aktualisiert und deployt werden konnten. 
Auch die Einbindung von SSL-Zertifikaten war dank NGINX relativ straight forward.
Jedoch birg das Setup auch Nachteile: es laufen immer mehrere Container, meist braucht man Extras wie Docker Compose um 
die Dinge sinnvoll wartbar zu gestalten. Warum also die UI nicht mit dem Backend ausliefern? Warum also nicht?

### Schritt 1: Aus Zwei mach Eins

Als erstes habe ich also mein Frontend ins Backend-Projekt geschoben. Statt Frontend habe ich das Verzeichnis allerdings ui genannt.

![Projektstruktur](../../posts/2020-09-17/structure.png)

### Schritt 2: Der lokale Build

Damit die UI das lokale Backend auch erreicht sollte — sofern nicht ohnehin schon geschehen — ein Proxy 
in der package.json hinterlegt werden. Natürlich mit der entsprechenden Port-Konfiguration. In meinem Fall :8000
```json
{
  "name": ..
  "proxy": "http://localhost:8000",
  "dependencies": {
    ... 
  }
}
```

Startet man jetzt das Backend und Frontend separat, müsste alles wie bisher funktionieren:

```shell
go run .
npm run start
```

### Schritt 3: Das Dockerfile

Sollte das soweit geklappt haben, möchte man das Artefakt nun wahrscheinlich als Container verpacken und dann 
verteilen oder deployen. Ich muss zugeben, das Dockerfile richtig zu bauen hat mich ein bisschen Zeit gekostet.
Umso ratsamer, den Stand hier zu teilen. Wie immer gilt auch hier: Feedback ist willkommen!

```dockerfile
FROM node:alpine as frontend
RUN mkdir /build
COPY ui /build/ui
WORKDIR /build/ui
RUN npm install
RUN npm run build

FROM golang:alpine as backend
RUN mkdir /build
ADD . /build/
WORKDIR /build
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-extldflags "-static"' -o main .

FROM scratch
COPY --from=frontend /build/ui/build /app/ui/build
COPY --from=backend /build/main /app/
WORKDIR /app
CMD ["./main"]
```

Okay, was passiert hier?

Der erste Block kopiert das ui-Verzeichnis in den Container und baut ein Produktions-Bundle der React-Anwendung. 
Darauf hin wird das Backend gebaut. Im dritten Schritt erfolgt dann die Vereinigung. 
Das Backend — hier die Datei main — wird aus dem Backend-Build-Container in das app-Verzeichnis kopiert. 
Danach passiert das gleiche mit dem build-Verzeichnis der React-Anwendung. So sollte im neu erzeugten Container 
die main und daneben das ui-Verzeichnis liegen. Mit dem build-Verzeichnis im Petto.

Daraufhin wird die Anwendung gestartet und: Tada! Frontend, Backend, ein Container.


### Ausblick

Jetzt wo das geklappt hat, werde ich versuchen die Verbindung des http-Servers des Go-Backends mit TLS abzusichern. 
Vermutlich werde ich einfach versuchen, die LetsEncrypt-Zertifikate der Maschine in den Container zu mounten und 
dann dem http-Server unterzuschieben. Ich werde berichten. Ciao!