---
title: "Today I Learned #1 - Testcontainers"
date: 2019-11-15
tags: ["Java", "Testcontainers", "Today I Learned"]
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
    image: "covers/today-i-learned.png"
    alt: Cover Image
    relative: false
    hidden: false
---

###3 Use Case

Man stelle sich folgendes Szenario vor: Ein Spring Boot Microservice, Unit Tests mit H2 In-Memory Datenbank, 
integrative Tests dann produktionsnah mit einer PostgreSQL DB.

Kommt man nun auf die Idee ein Feature zu benutzen, das — nennen wir es mal Postgres-nativ ist — stößt man 
in den Unit Tests schnell an die Grenzen der H2-Datenbank. In diesem Fall bei Benutzung des JSONB-Datentyps. 
Dieser erlaubt es, ähnlich wie in einem Document Storage, JSON-Objekte in einer Spalte zu persistieren. 
Dabei sind alle Felder des Objektes such-, änder- und indizierbar. Also ein Kompromiss zwischen klassischer 
relationaler Datenbank und NoSQL-Ansätzen à la MongoDB. Sounds fair!

### Das Problem

Nun lässt sich der Datentyp JSONB allerdings nicht mit H2, HSQLDB oder analogen In-Mem Datenbanken nutzen. 
Eine lauffähige embedded Postgres gibt es auch nicht. 
Bleibt also nur der Verzicht von Datenbanktests auf Unit Test Ebene (👎🏻 ) oder das Nutzen einer 
Standalone Postgres Instanz auf dem Entwicklerrechner (noch mehr 👎🏻). Ooooooder...

### Die Lösung

... man nutzt Testcontainer!

Statt also eine In-Memory Datenbank zu erzeugen, wird für den Test ein Container mit einem definierten Image gestartet. 
Die Anwendung wird dann, beispielsweise über die application.yaml, entsprechend für diesen Container konfiguriert. 
Was braucht man dafür?

Als erstes die entsprechenden Dependencies in die build.gradle:

````groovy
testImplementation("org.testcontainers:testcontainers:1.12.3")
testImplementation("org.testcontainers:postgresql:1.12.3")
````

Dann habe ich ein Test-Profil angelegt, dass die Verbindung für den Unit Test entsprechend 
in der application-test.yml konfiguriert:

````yaml
spring:
  datasource:
    url: jdbc:tc:postgresql:9.6.3://localhost/db
    username: user
    password:
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
````

In der Test-Klasse wird dann nur noch das Test-Profil gesetzt und schon sollte beim Start des Unit Tests das 
Postgres-Image geladen und als Container gestartet werden.

````java
@SpringBootTest
@ActiveProfiles("test")
class Test {
    ...
}
````

Das Image liegt dann in guter alter Docker-Manier übrigens auf dem Client und muss natürlich 
beim nächsten Run nicht wieder gezogen werden.

### Fazit

Mit Hilfe von Testcontainern lassen sich relativ schnell und einfach produktionsnahe Bedingungen für 
Unit Tests erzeugen. Das dabei die Laufzeit in Mitleidenschaft gezogen wird sollte Jedem bewusst sein. 
Aber einen Tod muss man halt sterben.