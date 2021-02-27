---
title: "Side Projects #2 - Annabingo"
date: 2020-11-25
tags: ["Annabingo", "Side Projects", "Golang", "React", "Docker", "Kubernetes"]
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

Neben [Annapoker](https://annapoker.de) habe ich in den letzten Monaten der Pandemie mehrere Ideen zur besseren 
verteilen Zusammenarbeit umgesetzt. Heute geht es um [Annabingo](https://annabingo.de)!
Auch das Projekt findet man natürlich in meinem [GitHub Repository](https://github.com/mz47/annabingo).

### Der Use Case

![Bingo](https://64.media.tumblr.com/12e04ccc852094d887b3178fdd097931/tumblr_nxidzlnBgT1tnr350o2_400.gif)

Was ist die aktuelle Lieblingsbeschäftigung aller, die im Homeoffice verweilen? Meetings.
Wie finden diese Meetings statt? Teams. Zoom. Skype.
Sind diese Meetings immer und in gänze spannend? Naja...

Und genau da kommt Annabingo ins Spiel.

Idee war es eine simple Bingo-Implementierung umzusetzen, die sowohl als Bullshit-Bingo für schier endlose 
Meeting-Marathons funktioniert, als auch als klassischer Ice Breaker für die nächste Team Runde.

Das Prinzip ist ganz einfach: man erstellt ein neues Bingo-Feld, füllt alle 20 Kästchen mit Phrasen oder Worten
und teilt den Link. Jeder erhält natürlich eine andere Reihenfolge. Wer zuerst vier Kästchen markiert hat, gewinnt.

### Die Technik

Der Use Case klingt schonmal simpel. Die Technik darunter ist es auch. Wieder mal lag eine Verbesserung meiner
Frontend-Skills im Fokus.

#### Backend

Was habe ich wohl im Backend gemacht. Wahrscheinlich Go, oder? Bingo!
Nichts außergewöhnliches zu berichten.

#### Frontend

Auch diesmal bin ich bei React gelandet. Ich muss ja gestehen, dass ich eigentlich Svelte ausprobieren wollte.
Eigentlich. Nach einigem Hin und Her beim Routing, wurde mir das ganze Framework-Geraffel aber zu kompliziert.
Da lobe ich mir doch den modularen Aufbau von React und das gute Ökosystem - mittlerweile, fairerweise.

Im Gegensatz zu Annabingo habe ich hier nicht Grommet als UI Framework benutzt, sondern Bootstrap. 
Ich wollte schon länger die React-Integration testen und war positiv überrascht. Alles klappt wie gewohnt.

#### Datenbank

Die Daten speichere ich aktuell BuntDB. Dies ist ein embedded Key-Value Store geschrieben in Go, ganz simpel. 
Das funktioniert gut, ist schnell genug und reicht für die aktuelle Auslastung. Klar könnte man auch einen Redis-Cluster
analog zu Annapoker verwenden, die Daten aus nach der TTL ausphasen lassen, das war mir bisher aber zuviel Aufwand.

Warum überhaupt Persistenz, wird man sich vielleicht fragen. Naja, ab und zu macht es scheinbar Sinn, ein passendes
Bingo-Spielfeld am Start zu haben. Vor allem wenn man den Ice Breaker in mehreren Runden verwendet.

#### Reverse Proxy

Auch hier benutze ich den Nginx als Reverse Proxy, ich mag die simple Konfiguration, die Erweiterbarkeit und 
die Performance. Daumen hoch!

#### Also nochmal kurz zusammengefasst:

- React und Bootstrap im Frontend
- Go im Backend
- BuntDB als Persistenz
- Nginx als Reverse Proxy

### Lessons Learned

Ob ich auch diesmal was gelernt habe? Eigentlich nicht. 
Das war eine schöne Fingerübung, die mich ein oder zwei Abende beschäftigt hat. Spaßig war's dennoch.

### Fazit

Annabingo hat Spaß gemacht. Ein kleines, nützliches Tool, das mir schon das ein oder andere Meeting weniger
Anstrengend vorkommen ließ. Leider denke ich viel zu selten daran, es einzusetzen. 
Aber vielleicht hast Du da ja den perfekten Einsatzzweck zu? Oder Feedback? Ich freue mich. Bis bald!