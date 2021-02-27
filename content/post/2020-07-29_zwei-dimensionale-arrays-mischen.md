---
title: "Zwei-dimensionale Arrays mischen"
date: 2020-07-29
tags: ["Golang", "Strukturen", "Algorithmus"]
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
    image: "https://images.unsplash.com/photo-1462927114214-6956d2fddd4e?ixid=MXwxMjA3fDB8MHxzZWFyY2h8MTN8fHNob2VzfGVufDB8fDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=900&q=60"
    alt: Cover Image
    relative: false
    hidden: false
---

Heute stand ich vor der Aufgabe ein zwei-dimensionales Array, gefüllt mit Strings, zu mischen. 
Hintergrund: in dem Array stecken Fragen, die bei jedem Request in anderer Reihenfolge zurückgegeben werden sollten.

Nach kurzer Recherche fand ich einen passenden Algorithmus: den Fisher-Yates Shuffle. 
In erster Ausbaustufe eher für eindimensionale Arrays gedacht, habe ich es aber schnell adaptieren können. 
Natürlich in Go für meine kleine Go-API.

````go
func (a *App) shuffle(bingo *[4][4]string) *[4][4]string {
  for i := len(bingo) - 1; i > 0; i-- {
    for j := len(bingo[i]) - 1; j > 0; j-- {
      m := rand.Intn(i + 1)
      n := rand.Intn(j + 1)

      temp := bingo[i][j]
      bingo[i][j] = bingo[m][n]
      bingo[m][n] = temp
    }
  }
  return bingo
}
````

Entgegen genommen wird ein Pointer auf ein String-Array mit jeweils vier Werten. 
Folglich werden die Werte dekrementell durchgetauscht und nach erfolgreichem Durchlauf zurückgegeben.

Das ganze Projekt findet sich übrigens in meinem GitHub Repository annabingo-backend.