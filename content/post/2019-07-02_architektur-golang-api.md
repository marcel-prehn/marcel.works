---
title: "Architektur einer Golang API"
date: 2019-07-02
tags: ["Golang", "Architektur", "API"]
author: "marcel"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
disableHLJS: true
disableShare: false
searchHidden: true
cover:
    image: "https://user-images.githubusercontent.com/567298/55478904-236e9200-561d-11e9-9382-f31b25a9ae03.png"
    alt: Cover Image
    relative: false
    hidden: false
---

Long time no hear. Heute melde ich mich mit einem Thema zurück, dass mir immer mal wieder Kopfschmerzen bereitet. 
Kurzzeitgedächtnis sei dank.
Kurzzeit-was?

![Meme hä?](../../posts/2019-07-02/meme.png)

Aber genau dafür dient mir ja dieser Blog: als Dev Journal. 
Also, heute geht es um Go. Genauer gesagt, um das kleine 1x1 der Architektur einer API, 
bzw. eines kleinen Services, der eine API darstellen soll.

### Der Use Case

Das ist relativ simpel: ein Proxy, der mit der Todoist-API telefoniert und mir meine Projekte, Tasks, Dues 
etc. zurück liefert. Die Daten verarbeite ich dann im Service und stelle sie aggregiert meinem Frontend (to be) 
zur verfügung. Klingt ein bisschen hanebüchen, ist aber unumgänglich. 
Warum? Na weil ich meine Todos im Kanban-Style verwalten möchte.

### Das Grundgerüst

Ich arbeite mit Go in Version 1.12.3 unter MacOS mit VisualStudio Code. 
Die Struktur des Projektes besteht aus meiner main.go und einem app-Package. 
Innerhalb des app-Packages gibt es ein handler- sowie ein model-Package. 
So ist es relativ übersichtlich und dennoch nach Concern getrennt.

![Das Grundgerüst](../../posts/2019-07-02/structure.png)

### Go Projekt Bootstrap

Mir persönlich fällt es immer schwer, nach monatelangem Java-Enterprise-Coding mit Spring 
entsprechend umzudenken und nicht bei Projektbeginn schon Controller-, Entity- und Service-Packages anzulegen. 
Durch den rudimentären Aufbau mit app-, model- und handler-Package schleicht sich allerdings 
nicht ganz so viel Boilerplate ins Projekt. #IMHO

### Die Implementierung

Wie geht es weiter, wenn die Struktur steht? Ich persönlich bastle als erstes die Modelle, 
sprich in diesem Fall mein Project- und Task-Struct.

````go
package model

type Project struct {
	ID   int64  `json:"id"`
	Name string `json:"name"`
}

type Task struct {
	ID        int64  `json:"id"`
	ProjectID int64  `json:"project_id"`
	Content   string `json:"content"`
}
````

Stehen die Modelle, geht es weiter mit dem Handler. Hier findet der eigentliche API Call gegen die 
Todoist-Schnittstelle statt. In meinem Fall stelle ich die Anfrage, parse die Antwort und gebe mit der 
Funktion RenderJSON wieder JSON zurück. Als Router benutze ich übrigens den httprouter von Julien Schmidt.

````go
//handler.go
package handler

import (
	"encoding/json"
	"io/ioutil"
	"log"
	"net/http"

	"github.com/mz47/todoist-go/app/model"
	"github.com/mz47/todoist-go/task"

	"github.com/julienschmidt/httprouter"
)

func GetAllProjects(w http.ResponseWriter, r *http.Request, p httprouter.Params) {
	client := http.Client{}
	request, error := http.NewRequest("GET", "https://beta.todoist.com/API/v8/projects", nil)
	if error != nil {
		log.Fatal("Error while sending request to todoist api", error)
	}
	request.Header.Add("Authorization", "Bearer "+GetAPIKey())
	resp, error := client.Do(request)
	if error != nil {
		log.Fatal("Error while receiving response from todoist api", http.StatusInternalServerError)
	}
	body, error := ioutil.ReadAll(resp.Body)
	if error != nil {
		log.Fatal("Error while parsing json response", error)
	}
	defer resp.Body.Close()
	projects := []model.Project{}
	json.Unmarshal(body, &projects)
	ReturnJSON(w, projects, http.StatusOK)
}
````

Hervorzuheben sind die Funktionen ReturnJSON sowie GetAPIKey. 
Beide Methoden liegen in common.go im selben Package wie der Handler. 
Daher entfällt der Aufruf über den Qualifier.

```go
//common.go
package handler

import (
	"encoding/json"
	"log"
	"net/http"
	"os"
)

func ReturnJSON(w http.ResponseWriter, object interface{}, statusCode int) {
	payload, error := json.Marshal(object)
	if error != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte(error.Error()))
		return
	}
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(statusCode)
	w.Write(payload)
}

func GetAPIKey() string {
	apiKey := os.Getenv("TODOIST_API_KEY")
	if apiKey == "" {
		log.Fatal("Error while getting todoist api key env var")
	}
	return apiKey
}
```

So! Model: check! Handler: check! Utility-Methoden: check! Jetzt wird es Zeit, alles zusammen zustecken. 
Dazu definiere ich ein App-Struct und binde den Router ein. 
In der Initialisierung des Routers dann die Verbindung zwischen Request und dem entsprechenden Handler-Aufruf. 
Hier: ein GET um alle Projekte abzuholen. Danach nur noch die Run-Methode um den HTTP-Server zu starten und das wars! 
Easy as that!

```go
// app.go
package app

import (
	"log"
	"net/http"

	"github.com/julienschmidt/httprouter"
	"github.com/mz47/todoist-go/app/handler"
)

type App struct {
	Router *httprouter.Router
}

func (a *App) Init() {
	a.Router = httprouter.New()
	a.initRouter()
}

func (a *App) initRouter() {
	a.Get("/projects", handler.GetAllProjects)
	a.Get("/projects/:projectID", handler.GetTasksByProject)
}

func (a *App) Get(path string, handle func(w http.ResponseWriter, r *http.Request, ps httprouter.Params)) {
	a.Router.GET(path, handle)
}

func (a *App) Run(host string) {
	log.Fatal(http.ListenAndServe(host, a.Router))
}
```

Die main.go sieht nun relativ karg und unspektakulär aus. 
Lediglich die Instanziierung der app-Komponente, die Initialisierung dieser und der 
obligatorische run()-Aufruf um den Service zu starten. 
Fertig!

```go
//main.go
package main

import (
	"github.com/mz47/todoist-go/app"
)

func main() {
	app := &app.App{}
	app.Init()
	app.Run(":8080")
}
```

### Die Moral der Geschichte

Viele Wege führen nach Rom. Für mich persönlich ist dieser hier aufgezeigte allerdings der charmanteste. 
Wenig Boilerplate, wenig Struktur. #leancoding Dennoch muss ich nach ein paar Go-losen Wochen auch wieder nachdenken 
und -lesen, wie es eigentlich ging.

Wie sich das ganze anfühlt, wenn das Projekt immer weiter wächst, wird sich noch herausstellen. 
Aber für den kleinen Microservice für nebenan und den Heimgebrauch macht es bisher einen guten Eindruck. 
Gerne lasse ich mich eines Besseren belehren. Bis dahin!
