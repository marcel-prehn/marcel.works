---
title: "RethinkDB #2 - Count"
date: 2020-08-27
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

Wie schon in meinem letzten Beitrag zum Thema RethinkDB RethinkDB #1 — Update, hier Teil #2. 
Diesmal zähle ich Datensätze, die ein bestimmtes Kriterium erfüllen. 
Wieder in Go, wieder auf einer RethinkDB, wieder hatte ich keine Ahnung.

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

Daran hat sich zum letzten Mal nichts geändert. Wieder ein Session-Objekt mit einer Liste bestehend aus Usern. 
Jetzt wollte ich allerdings wissen, wieviele User es in einer bestimmten Session gibt, 
die noch kein Voting abgegeben haben; sprich das Attribut voting ist 0. Direkt denkt man an

```sql
SELECT COUNT(*) FROM SESSION WHERE SESSION_ID = X AND VOTING = 0
```

Klar, in den 90ern wäre das richtig gewesen. Aber da gab es ja auch noch keine Document Stores. Also, glaube ich.

Wie bewerkstelligt man das also auf einer RethinkDB?

### Die Query

```go
r.DB(db).Table(tableSessions).
   Get(sessionId).
   Field("users").
   Field("Voting").
   Count(func(v r.Term) r.Term {
      return v.Eq(0)
   }).
   Run(s.Session)
```

Zur Erklärung: Wir nehmen die Datenbank, wählen die Tabelle aus und selektieren die Session mit der entsprechenden SessionId.
Daraufhin holen wir uns das Feld users. Aus den users dann jeweils das Feld Voting. Und zählen.
Die Zählfunktion erwartet eine Funktion, die einen Term als Parameter erwartet und einen Term zurück gibt. 
Dieser Rückgabewert ist die eigentliche Operation: der Vergleich des Feldes Voting bzw. v auf Gleichheit mit 0. 
Ist die Bedingung erfüllt, wird gezählt.

Das Result dieser Query ist dann die korrekte Anzahl. Ratet mal welcher Datentyp da zurück kommt? 
Na klar, ein 64bit Float! Wer zählt heutzutage nicht in Fließkommazahlen? Das ist hip!

### Fazit

Verglichen mit dem Update aus Teil #1 war das schon fast einfach. 
Leider findet man nur dazu auch relativ wenig im Internet. Daher hier meine Dokumentation dazu.