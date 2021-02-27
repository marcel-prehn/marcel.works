---
title: "RethinkDB #1 - Update"
date: 2020-08-26
tags: ["Golang", "RethinkDB"]
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
    image: "https://images.unsplash.com/photo-1555532538-dcdbd01d373d?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=2631&q=80"
    alt: Cover Image
    relative: false
    hidden: false
---

Aktuell entwickle ich eine Webanwendung, die über Websockets mit dem Backend kommuniziert.
Auf der Suche nach einer passenden Datenbank dafür, bin ich neben den üblichen Verdächtigen wie
MongoDB, CouchDB, … über RethinkDB gestolpert. Die Marketing-Texte laßen sich gut, einen passenden Go-Treiber gabs auch.
Ich war überzeugt davon, das die neue Datenbank mal auszuprobieren. Eine nette Web-UI frei Haus
und die einfache Möglichkeit, das Ding im Cluster zu betreiben waren schöne Goodies.

Das Go-Module war schnell installiert, der Docker-Container gestartet und die Verbindung rasch aufgebaut.
Soweit so gut! Auch das Insert funktionierte wie erwartet. Auf erste Probleme bin ich dann aber beim partiellen
Update eines Datenfeldes gestoßen. Nach vielen Experimenten über die Web-UI und lange Google-Sessions,
bin ich dann doch drauf gekommen. Und diese Erkenntnis wollte ich unbedingt mit der Nachwelt teilen.

Es ging um folgendes…

### Die Datenstruktur

```json
{
    "Id": "fe049091-2287-4949-bb22-1a9eca6838b7",
    "Users": [
        {
            "uuid": "f7740b68-2989-4e77-b551-d44e774ac2bb",
            "username": "yolo",
            "voting": 0
        },
        {
            "uuid": "a3bb02fb-c340-4615-bd6a-006b7aace39e",
            "username": "marci",
            "voting": 0
        },
        {
            "uuid": "a03d5fd7-3c8b-460a-a65a-eeed21ce13fb",
            "username": "QA",
            "voting": 0
        },
        {
            "uuid": "f7740b68-2989-4e77-b551-d44e774ac2bb",
            "username": "yolo",
            "voting": 0
        }
    ]
}
```

Es geht also um ein Objekt (session), das eine Liste von weiteren Objekten (users) als Datenfeld besitzt.
Was ich nun wollte war, das Attribut voting eines bestimmten users zu überschreiben.
Also in der Theorie:

```javascript
DB.sessions.get("uuid").users("uuid").voting = 2
```

Aber falsch gedacht! Das eigentliche Query war viel unintuitiver.

### Die Query

```go
r.DB(db).Table(tableSessions).
   Get(sessionId).
   Update(
      map[string]r.Term{"users": r.Row.Field("users").Map(func(u r.Term) r.Term {
         return r.Branch(
            u.Field("Uuid").Eq(user.Uuid),
            u.Merge(map[string]int{"Voting": user.Voting}),
            u,
         )
      })},
   ).
    RunWrite(s.Session)
```

Gut, die Dokumentation der RethinkDB-API für den Go-Treiber war nur rudimentär vorhanden,
die Leute die es benutzen nur überschaubar, aber sowas? Hui! Ein passender Feature Request,
um Felder spezifisch zu aktualisieren, existiert übrigends schon länger.

Aber jetzt mal zur Erklärung:

Wir holen uns die Datenbank, klar. Daraufhin die Tabelle und den entsprechenden Eintrag darin mit Hilfe der SessionId.
Haben wir den Datensatz, rufen wir auf dieser Selektion die Update-Funktion auf.
Innerhalb des Updates holen wir uns das Feld users und mappen dessen Inhalt.
Die anonyme Funktion nimmt einen Term entgegen und liefert einen zurück. Das Ergebnis ist ein Branch-Kommando,
dass den entsprechenden User mit der gegebenen UserId findet.
Haben wir den User erstmal, machen wir einen Merge und überschreiben das Attribut voting.

Ganz easy!

### Fazit

Das ganze Update-Statement hat mich einen halben Abend gekostet, woran das gelege hat..
Mittlerweile nutze ich übrigens Redis statt RethinkDB. Flüchtiger Speicher passt einfach besser zu meinem Use Case.
Aber das nur nebenbei.

Ich hoffe, falls Jemand mal in die selbe Problemstellung läuft, hiermit geholfen zu haben.