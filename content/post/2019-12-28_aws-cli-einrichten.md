---
title: "AWS CLI einrichten"
date: 2019-12-29
tags: ["AWS", "CLI", "DevOps"]
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
    image: "https://images.unsplash.com/photo-1516259762381-22954d7d3ad2?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=850&q=80"
    alt: Cover Image
    relative: false
    hidden: false
---

Irgendwann kommt man an den Punkt, an dem man es Leid ist alle Services über die AWS Console einzurichten. 
Dann ist der Zeitpunkt gekommen, von der UI auf die Konsole umzusteigen. 
Und wie man das konfiguriert, zeige ich hier als Beispiel unter MacOS ...

### 1. AWS Zugriffsschlüssel erzeugen

Als erstes sollte man einen Access Key und entsprechendes Secret in seinem AWS Konto anlegen. 
Damit ist es später möglich, sich über das CLI zu authentifizieren und zu autorisieren. 
Dazu einmal in die AWS Console, klick auf den User oben rechts, dann auf Meine Sicherheitsanmeldeinformationen.

![Konto](../../posts/2019-12-29/aws-1.png)

und schlussendlich den Zugriffsschlüssel erstellen.

![Zugriffsschlüssel generieren](../../posts/2019-12-29/aws-2.png)

![Zugriffsschlüssel generieren](../../posts/2019-12-29/aws-3.png)

### 2. Python3 installieren

Hat man den Key und das Secret angelegt, geht es an die Installation des AWS CLI. 
Dieses benötigt Python. Mein MacBook wurde zwar mit Python ausgeliefert, jedoch in Version 2.7. 
Um in Zukunft Feature-kompatibel zu sein, installiere ich Python3 über Homebrew nach.

````shell
brew install python
````

Danach verlinke den python-Befehl mit python3 über ein Alias in meinem Bash-Profil, 
damit die Installationsroutine der AWS CLI die aktuelle Version statt der 2.7 verwendet. 
Das selbe mache ich für pip. Das Bash-Profil findet man in der regel im Home-Verzeichnis.

````shell
vim ~/.zshrc
````

Daran füge ich die Alias-Kommandos am Ende der Datei hinzu.

````shell
alias python=/usr/local/bin/python3
alias pip=/usr/local/bin/pip3
````

Nach einem Neustart der Konsole oder einer Aktualisierung des Profils, 
sollten das python- bzw. pip-Kommando auf die Version 3.x zeigen.

````shell
source ~/.zshrc
python --version
Python 3.7.6
````

### 3. CLI installieren

Sollte das Installieren von Python geklappt haben, kann man nun die AWS CLI installieren. In meinem Fall die Version 2. 
Dazu kann man sich am Besten am Leitfaden von Amazon orientieren und die Installationsroutine herunterladen, 
entpacken und ausführen.

````shell
curl "https://d1vvhvl2y92vvt.cloudfront.net/awscli-exe-macos.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
````

Die Installation lässt sich danach einfach mit dem aws2-Befehl prüfen.

````shell
aws2 --version
aws-cli/2.0.0dev2 Python/3.7.4 Darwin/18.7.0 botocore/2.0.0dev1
````

Aus Gründen der Gemütlichkeit habe ich anschließend — analog zum Vorgehen bei Python — ein Alias für den aws2-Befehl angelegt, 
der aws auf aws2 mappt. Dazu wieder im Bash-Profil im Home-Verzeichnis eine Zeile anfügen.

````shell
alias aws=/usr/local/bin/aws2
````

### 4. CLI konfigurieren

Wenn das alles geklappt hat, muss nun nur noch das CLI mit dem AWS-Konto verknüpft werden. 
Dazu benötigen wir endlich den Access Key und das Secret aus dem 1. Schritt. 
Am einfachsten lässt sich das CLI über den configure-Befehl einrichten.

`````shell
aws configure
`````

Dabei folgt man dem Wizard, der nacheinander Access Key, Secret, Region und Output-Format erfragt. 
Als Default für Region und Output habe ich us-east-1 und yaml gewählt. Das ist aber persönlicher Gusto.

Die Konfiguration findet man übrigens im Home-Verzeichnis unter .aws/ für nachträgliche Anpassungen.

### 5. Zugriff testen

Ob du wirklich richtig stehst, siehst du wenn … der folgende Befehl funktioniert:

````shell
aws iam list-roles
...
````

Zu sehen sollten das alle IAM-Rollen des AWS-Kontos sein. 
Schließen lässt sich die Anzeige übrigens via Ctrl-C oder q.