---
title: "Side Projects #1 - Annapoker"
date: 2020-11-20
tags: ["Side Projects", "Annapoker", "Golang", "Redis", "React", "RabbitMQ", "STOMP", "Docker", "nginx"]
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
    image: "https://i.ibb.co/WsfFP3j/Bildschirmfoto-2021-02-27-um-20-15-14.png"
    alt: Cover Image
    relative: false
    hidden: false
---

Ich dachte mir ich berichte einfach mal in einer neuen Serie von kleineren und größeren Projekten, 
die ich so nach Feierabend umsetze. Dabei liegt mein persönlicher Fokus meist auf dem Erlernen neuer 
Technologien oder Systeme. Aber ab und zu kommt tatsächlich was sinnvolles dabei raus. 
Zum Beispiel ![Annapoker](https://annapoker.de)! 
Das ![Repository](https://github.com/mz47/annapoker) zum Projekt findet man natürlich in meinem GitHub.

### Der Use Case

![Pokerface](https://i.imgur.com/KMus6br.gif)

Der aktuellen Situation geschuldet kommt es ja häufiger vor, dass immer mehr Menschen verteil miteinander arbeiten müssen. 
Vor allem in agilen Teams gab es da mit dem Ausbruch der Home-Office Offensive häufiger den Bedarf nach Tools zur Kollaboration. 
In meinem aktuellen Projekteinsatz kam es nach einigen Wochen zu Unmut über das bisher verwendete Planning Poker Tool. 
Warum also nicht eine charmante Alternative schaffen?

Für alle, die mit dem Begriff Planning Poker nichts anfangen können: 
beim Planning Poker, wird die Komplexität von Anforderungen relativ zu einander geschätzt. 
Anhand der Fibonacci Zahlen bewertet jedes Team-Mitglied eine umzusetzende Anforderung für sich. 
Mit Aufdecken aller Einschätzungen entsteht eine Diskussion über deren Zustandekommen und am Ende 
quasi ein Preis in der Einheit Story Points pro Anforderung.

### Die Technik

Was braucht es alles für einen Webservice, der zur unidirektionalen Schätzung dient? 
Und viel wichtiger: was wäre neuer, heißer Sch.. Kram, den man ausprobieren könnte?

#### Backend

Der Stack war schnell zusammen gedacht: Go für's Backend - klar! 
Ausprobieren wollte ich in diesem Zuge einmal zap als strukturierter Logger und fx als Dependency Injection Framework. 
Beide Projekte stammen aus dem Hause Uber.

#### Frontend

Um meine UI-Kenntnisse ein wenig zu schärfen, gibt's React im Frontend. 
Diese React Hooks wollte ich mir schon ewig mal anschauen. 
Für die grafische Aufbereitung habe ich mich für Grommet entschieden.
Bunte Farben, große Elemente, perfekt!

#### Transport-Layer

Für die Kommunikation kommen eigentlich nur WebSockets in Frage. 
Als Protokoll habe ich mich für STOMP entschieden. Ein passendes Node-Module und eine Go-Lib gibt's schon - passt! 
Und weil man ja theoretisch skalierfähig sein sollte, braucht man einen Broker für STOMP. 
Gut, dass RabbitMQ das kann. Also auch RabbitMQ in den Stack.

#### Datenbank

Was jetzt noch fehlt ist die Persistenz. Und da war es spannend. 
Für den MVP habe ich einen InMemory-Key-Value-Store genommen (BuntDB). 
Für Version 1.0 habe ich allerdings auf eine RethinkDB umgestellt. 
as lief, wie man meinen letzten Blog-Posts entnehmen kann, nur so semi-gut. 
Da ich für die eigentliche Poker-Session aber sowieso keinen dauerhaften Datenspeicher brauche, habe ich mich für Redis entschieden. 
Da eine Schätzrunde in der Regel nicht länger als zwei Stunden läuft und die Ergebnisse danach irrelevant sind, 
fand ich es sinnvoll einen Cache zu benutzen und den eigentlichen Datensätzen eine Time-To-Live von acht Stunden einzuräumen. 
Hat auch den Vorteil, das die Datenbank nie voll läuft. Um eventuelle Ausfälle abfedern zu können, 
läuft der Redis-Cache im Cluster mit insgesamt 4 Nodes. Nötig ist es nicht, aber man weiß ja nie.

#### Reverse Proxy

Als Reverse Proxy benutze ich einen Nginx. 
Die Services laufen alle als Single Instance Docker Container auf einem VPS, orchestriert mit docker-compose.

#### Additum

Für das Logging und Tracing, sowie die Systeminformationen des Hosts benutze ich übrigens Datadog - sehr zu empfehlen! 
Die Nginx und Redis Integrationen sind schnell installiert und super übersichtlich.

#### Also nochmal kurz zusammengefasst:

- React mit Grommet als UI-Bibliothek im Frontend
- Go im Backend 
- Kommunikation mit STOMP via WebSocket
- RabbitMQ als Broker
- Redis als flüchtiger Datenspeicher
- Nginx

### Lessons Learned

Was habe ich gelernt... zum Glück so einiges!

#### Punkt 1: RethinkDB

Die RethinkDB hat mich viele Stunden Research und Development gekostet. 
Meiner Meinung nach noch nicht ganz ausgereift und an der ein oder anderen Stelle etwas zu komplex gedacht, 
aber definitiv vielversprechend. Vielleicht komme ich ja irgendwann nochmal dazu, die Streaming-Qualitäten zu testen.

#### Punkt 2: Frontend mit dem Backend gemeinsam ausliefern

Das war zwar schön gedacht, hat aber so einige Pitfalls mitgebracht, die es mir am Ende des Tages nicht wert waren zu lösen. 
So hatte ich beispielsweise Probleme mit dem Routing. Im Frontend habe ich dazu den BrowserRouter benutzt. 
Bei einem Redirect sprang natürlich nicht nur der an, sondern auch die Middleware - in meinem Fall Gin - im Backend. 
Nervig. Also kurzerhand auf den HashRouter umgestellt. Hat aber auch nicht geholfen. 
Jetzt hat sich der useEffect-Hook geweigert, die Seite nach einem Redirect zu laden. 
Der Lifecycle hat also nicht mehr funktioniert.

#### Punkt 3: Redis Cluster lokal auf dem Mac

Scheinbar auch nicht so einfach. Nach ein bisschen Recherche und scheinbar endlosen Shell-Scripts die 
Abhilfe schaffen sollten, habe ich statt des nativen Clusterings auf einen Master-Slave-Betrieb umgestellt. 
Das war easy. Über das genaue Vorgehen werde ich separat berichten.

#### Punkt 4: Ohne Backend geht's doch nicht (so einfach)

Im ersten Wurf von ![Annapoker](https://annapoker.de) habe ich noch probiert, die Clients sich untereinander synchronisieren zu lassen. 
Ohne den Einsatz eines Backends als zentrales Hub. Das hat anfangs auch gut funktioniert, 
jedoch wurde der Code schnell sehr komplex. 
Um das zu beheben (und weil es eine passende STOMP-Lib für Go gibt) kam also doch ein Backend dazu.

#### Punkt 5: fx hilft das Projekt zu strukturieren

Mir hat es geholfen das Backend ordentlich zu strukturieren, als ich dazu von fx gezwungen wurde. 
Gerade in Go hat man ja relativ viele Freiheiten, was das Packaging angeht. 
fx hilft da, in ordentlichen Mustern zu denken, damit die DI funktioniert. 
Längere Kompilierzeiten sind mir übrigens nicht aufgefallen. Insgesamt also ein Mehrwert für mich. 
Das Thema ist definitiv mal einen eigenen Blog-Post wert.

### Fazit

Das Projekt hat viel Spaß gemacht, ich habe einiges gelernt. Vor allem die Vorzüge der 
Backend-Entwicklung verglichen mit dem Frontend. Spaß beiseite.
Beim Kunden kam ![Annapoker](https://annapoker.de) gut an, bis auf kleinere Startschwierigkeiten hat alles gut funktioniert. 
Was mich jetzt natürlich interessiert ist, wie sich die Anwendung unter Last verhält. 
Vielleicht kommt der große Ansturm ja noch. 
Feedback ist natürlich immer willkommen.