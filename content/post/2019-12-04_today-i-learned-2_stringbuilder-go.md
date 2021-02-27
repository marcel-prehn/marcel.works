---
title: "Today I Learned #2 - StringBuilder in Go"
date: 2019-12-04
tags: ["Golang", "Today I Learned"]
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
    image: "https://images.unsplash.com/photo-1593600269510-0816682e80da?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=80"
    alt: Cover Image
    relative: false
    hidden: false
---

Ab und zu gerät man in die Bredoullie, einen laaaangen String aus mehreren Substrings zu konkatinieren. 
Das kann man entweder durch Addition machen.. oder mit einem StringBuilder.

Wie auch in anderen Highlevel-Sprachen wie C# oder Java, gibt es auch in Go einen StringBuilder der das 
Zusammensetzen und anschließende Ausgeben einer langen Zeichenkette unterstützt.

Wie das geht sieht man hier:

````go
sb := string.Builder{}
sb.WriteString("Langer")
sb.WriteString("String")
println(sb.String())
````

Zu erst erzeugt man sich eine neue Instanz des StringBuilders aus dem strings-Package. 
Angehangen werden neue Zeichenketten mit WriteString. 
Das eigentliche Konkatinieren findet dann mit dem Methodenaufruf ToString statt und 
gibt die zusammengesetzte Zeichenkette zurück. 
Ganz einfach.