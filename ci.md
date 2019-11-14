Documentation of the installation


# Jenkins Installation
Jenkins von https://jenkins.io/download/ herunterladen.

Zip Datei entpacken und .msi Installation ausführen. Einfach durchklicken und ausführen.

Nach Installation Browser öffnen und http://localhost:8080 eingeben.

Passwort aus der Datei gemäss Pfad auslesen und eingeben
![](/images/jenkins_01.png)

Install suggested plugins auswählen. Die Installation dauert einige Minuten
![](/images/jenkins_02.png)

Benutzer angeben (optional). Ansonsten unten auf Continue as Admin.
![](/images/jenkins_03.png)

Hier URL und Port anpassen (optional).
![](/images/jenkins_04.png)

Start using Jenkins klicken.


# Vagrant Build
Auf "Element anlegen" gehen
![](/images/jenkins_05.png)

Name in das Feld eingeben und "Free Style-Softwareprojekt bauen" auswählen und ok klicken
![](/images/jenkins_06.png)

Nach unten scrollen bis Buildverfahren. "Build-Schritt hinzufügen" klicken und "Shell ausführen" auswählen.
Anschliessend den change directory cmd eingeben wo das Vagrantfile befindet und auf der nächste zeile vagrant up eingeben.
![](/images/jenkins_07.png)

Dann auf Speichern klicken

Nun auf "Jetzt bauen" klicken
![](/images/jenkins_08.png)

Dann bei Build-Verlauf auf #1 klicken und anschliessend auf Konsoleausgabe gehen und prüfen ob exit Code 0 ist.
![](/images/jenkins_09.png)
![](/images/jenkins_10.png)
