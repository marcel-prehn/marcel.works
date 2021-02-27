---
title: "Side Projects #3 - Annaquiz"
date: 2020-12-05
tags: ["Annaquiz", "Side Projects", "Golang", "React", "Docker", "Kubernetes"]
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
    image: "covers/keyboard.png"
    alt: Cover Image
    relative: false
    hidden: false
---

Kommen wir zum dritten Projekt aus der Anna-Reihe: [Annaquiz](https://annaquiz.de).
Neben [Annapoker](https://annapoker.de) und [Annabingo](https://annabingo.de) steht auch Annaquiz ganz im Zeichen der Homeoffice-Zeit.

Auch hier wieder: Sourcen gibt's im [GitHub Repository](https://github.com/mz47/annabingo) des Vertrauens.

### Der Use Case

Analog zu [Annabingo](https://annabingo.de) dient auch [Annaquiz](https://annaquiz.de) als kleiner Ice Breaker für das nächste Meeting in großer Runde.
Oder die nächste Retrospektive. Oder. Oder. Oder.

Man erstellt ein neues Quiz, bestehend aus drei Fragen. Diese Fragen werden beantwortet und gespeichert.
Dann verteilt man den Link. Jeder Mitspieler bzw. Teilnehmer hat jetzt die Möglichkeit, jede Frage selbst zu beantworten.

Nachdem jeder die Fragen beantwortet hat, erhält man eine Übersicht, die zu jeder Frage eine beliebige Antwort der
Mitspieler ausgibt. Man kann dann versuchen zu erraten, wer wohl welche Antwort gegeben hat. Und dann je nach Gusto
die korrekten Antworten auflösen. Vielleicht erhält man so ein paar lustige Details über die eigenen Kollegen.

Soweit zur Idee.

### Die Technik

Ob es diesmal etwas neues, bahnbrechendes zu sehen gibt?

#### Backend

Nope. Leider nicht. Das Backend habe ich klassisch in Go geschrieben - das funktioniert einfach. Nichts neues hier.
Weitergehen!

#### Frontend

Auch im Frontend keine Revolution - ja nicht mal eine Evolution! Wieder React, wieder Grommet, wieder nichts spektakuläres.
Spaß gemacht hat es dennoch.

#### Datenbank

Auch hier nichts neues. BuntDB als Key-Value Store. Das langt eben. Und damit lässt es sich wunderbar einfach prototypen.

#### Reverse Proxy

Wieder Nginx. Wieder die selbe Konfiguration. Sorry!

#### Also nochmal kurz zusammengefasst:

- React und Grommet
- Go
- BuntDB
- Nginx

### Lessons Learned

Auch das nicht. Langweilig, oder? Naja, man muss das Rad ja nicht jedes Mal neu erfinden! :)

### Fazit

Kleines Tool, witze Idee, schnell gemacht. Und es funktioniert gut. Einfach mal ausprobieren und berichten.
Ich bin gespannt!