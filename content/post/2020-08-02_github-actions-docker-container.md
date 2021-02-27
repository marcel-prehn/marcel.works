---
title: "Mit GitHub Actions Docker Container bauen"
date: 2020-08-02
tags: ["GitHub", "GitHub Actions", "Docker", "CI", "CD", "Pipeline", "DevOps"]
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
    image: "https://images.unsplash.com/photo-1587654780291-39c9404d746b?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=850&q=80"
    alt: Cover Image
    relative: false
    hidden: false
---

Schon länger bietet GitHub die Möglichkeit eine CI/CD Pipeline aufzusetzen. Ähnlich der Möglichkeiten von GitLab CI. 
Nur statt der Ausführung auf dedizierten Maschinen a la GitLab Runner, laufen die Actions getauften Jobs direkt in der Cloud.

Für ein aktuelles Projekt, habe ich diese Actions mal ausprobieren wollen. 
Mein Use Case beschränkt sich momentan nur darauf, meine Go API in ein Docker Image zu packen und 
dann ins Docker Hub zu veröffentlichen.

### Die Konfiguration

Alles startet mit der Anlage der Konfiguration. Dazu wird ein .github benannten Verzeichnis im Root des Projektes angelegt. 
Darin habe ich noch ein workflows-Verzeichnis erstellt, um eventuelle Pipelines zu separieren.

![Projektstruktur](../posts/2020-08-02/structure.png)

Innerhalb der publish_docker.yml werden dann die nötigen Schritte zum Bauen und Veröffentlichen des Images definiert. 
In meinem Fall sieht die Konfiguration wie folgt aus:

````yaml
name: Publish Docker
on: push
jobs:
  build:
    name: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Login to DockerHub Registry
        run: echo ${{ secrets.DOCKERHUB_PASSWORD }} | docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin
      - name: Build the tagged Docker image
        run: docker build . --file Dockerfile --tag mz47/annabingo-backend:latest
      - name: Push the tagged Docker image
        run: docker push mz47/annabingo-backend:latest
````

Zur Erläuterung: abgesehen vom name ist definiert, dass die Pipeline bei jedem Push ins Repository ausgeführt. 
Alternative wären Ausführung bei einem Release zum Beispiel.

Danach wird ein Job definiert — build. Der Job besteht abgesehen vom name und der Angabe des Base-Images unter runs-on, aus Steps.

Jeder Step bildet dabei einen Arbeitsschritt ab. An dieser Stelle unterscheiden sich die Actions vom GitLab CI. 
Denn unter uses können bereits definierte Actions wiederverwendet werden. In meinem Fall nutze ich die Checkout-Action.

Die folgenden Steps sind eigentlich selbsterklärend: Login ins Docker Hub, Bauen und Taggen des Docker Images 
und anschließendes Publishing ins Docker Hub.

Auffallend sind noch die Referenzen auf die Secrets. Diese können einfach auf Repository-Ebene angelegt 
und in den Actions referenziert werden.

![GitHub](../../posts/2020-08-02/github.png)

### Die Pipeline

Ist alles richtig konfiguriert und ins Repository eingecheckt, ist die Pipeline im Actions-Tab sichtbar.

![GitHub Logging](../../posts/2020-08-02/github_actions.png)

Bei einem Klick auf den jeweiligen Run, ist auch die Konsolen-Ausgabe einsehbar. Vor allem für’s Debugging spannend.

![GitHub Logging](../../posts/2020-08-02/github_log.png)

### Fazit

Mit GitHub Actions ist eine CI/CD-Pipeline, vor allem in schon bestehende Repositories, schnell und komfortabel nachgerüstet. 
Ohne großen Overhead wie dem Aufsetzen eines Jenkins oder ähnliches.

Ich für meinen Teil werde mich fortan häufiger mit dem Thema beschäftigen und versuchen, auch komplexere Szenarien 
die das Deployment in einen Cluster beispielsweise abzubilden.