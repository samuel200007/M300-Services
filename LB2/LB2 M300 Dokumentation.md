# LB2 M300 Doku

##### Vorwissen:

Gar kein vorwissen

##### Befehle:

`docker run`
 startet einen neuen Container, unterstützt zahlreiche Argumente
 `docker ps`
 Übersicht über die aktuellen Container
 `docker images`
 Liste lokaler Images
 `docker rm`
 entfernt einen oder mehrere Container
 `docker rmi`
 löscht die angegebenen Images
 `docker start`
 startet gestoppte Container
 `docker stop`
 stoppt Container
 `docker kill`
 stopp Container sofort (Signal an Hauptprozess)
 `docker logs`
 Container Logs
 `docker inspect`
 Informationen zu Containern
 `docker diff`
 zeigt Änderungen am Dateisystem des Containers
 `docker top`
 Informationen zu laufendem Prozess im Container

## Wissensstand vor LB1

**Linux** 
 Erweiterte Linux kenntnisse durch ÜK-, TBZ- und Abteilungsprojekte. 
 Bash Experte und Linux Fan :)

**Git** 
 Zuvor noch nie mit Git gearbeitet. Nun habe ich dank diesem Modul mehr Erfahrung mit Git und kann es einwandfrei nutzen.

**Docker** 
 Noch gar kein Wissen nur 2 mal ein docker run Befehl ausgeführt, ohne  genau zu Wissen was gerade passiert. In die kalte Theorie geworfen  worden. Und sonst noch gar kein Praktisches Wissen.

## 

## Tools

**GitHUB** 
 Nach Anleitung von Herr Berger:
 *Github*  

1. Konto erstellen auf [www.github.com](http://www.github.com)
2. Bestätigung der E-Mail durch Verifications Link

*Repository*

1. Neues Repository Erstellen
2. Name meines Repositorys M300_Docker
3. Status des Repositorys auf Öffentlich stellen
4. README File erstellen, .md für die Richtige Markdown darstellung des Files

*Git Bash*

1. Git Herunterladen <https://git-scm.com/downloads>
2. Installation abschliessen und Git Bash öffnen
3. Konfigurieren per Bash Befehle:
    --> git config --global user.name "bmestry"
    --> git config --global user.email "[bryan.lo99la@gmail.com](mailto:bryan.lo99la@gmail.com)"

*Klonen des Repositorys*

1. Erstelltes Repository auf GitHub öffen. Auf Clone/Download drücken und den Link kopieren.
2. Bash öffnen und Ordner für dieses Repository oder allgemeiner Ordner für kommende Repositories erstellen.
3. Zum Directory wechseln und folgender Befehl ausführen

```
  git clone https://github.com/bmestry/M300_Bryan
  git status
```

### Kapitel 1

WordPress ist ein freies Content-Management-System. Es wurde ab 
2003 von Matthew Mullenweg als Software für Weblogs programmiert und 
wird als Open-Source-Projekt ständig weiterentwickelt. WordPress ist das
am weitesten verbreitete System zum Betrieb von Webseiten. Wordpress ist sehr simple aufgebaut. Man kann verschiedene Plugins und Vorlagen installieren um sich das erstellen der Webseite einfach zu machen.

### Kapitel 2

```
+---------------------------------------------------------------+
!                                                               !	
!    +-----------------------+    +------------------------+    !
!    ! Web-Server Wordpress  !    ! Datenbank-Server       !    !       
!    ! Port: 8000            !    ! Port: 3306           !    !       
!    ! Volume: /var/www/html !    ! Volume: /var/lib/mysql !    !       
!    +-----------------------+    +------------------------+    !
!                                                               !	
! Container                                                     !	
+---------------------------------------------------------------+
! Container-Engine: Docker                                      !	
+---------------------------------------------------------------+
! Gast OS: Ubuntu 16.04                                         !	
+---------------------------------------------------------------+
! Hypervisor: VirtualBox                                        !	
+---------------------------------------------------------------+
! Host-OS: Windows                             !	
+---------------------------------------------------------------+
! Notebook - Schulnetz 10.x.x.x                                 !                 
+---------------------------------------------------------------+
```

### Anleitung für den Betrieb:

##### Manuell aufsetzen:

Docker Compose installieren:

###### **Voraussetzungen:**

Docker Compose verwendet Docker Engine für jede sinnvolle Arbeit. Vergewissern Sie sich, dass Sie die Docker Engine entweder lokal oder remote installiert haben, abhängig von Ihrem Setup.

-   Auf Desktopsystemen wie Docker Desktop für Mac und Windows ist Docker Compose Bestandteil dieser Desktop-Installationen.

-   Installieren Sie auf Linux-Systemen zunächst den Docker für Ihr Betriebssystem, wie auf der Seite Docker abrufen beschrieben. Dann erhalten Sie Anweisungen zur Installation von Compose auf Linux-Systemen.

-   Informationen zum Ausführen von Compose als Benutzer ohne Rootberechtigung finden Sie unter Verwalten von Docker als Benutzer ohne Rootberechtigung.

##### **Installation**:

1) Führen Sie diesen Befehl aus, um die aktuelle stabile Version von Docker Compose herunterzuladen:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2) Anwenden von ausführbaren Berechtigungen auf die Binärdatei:

```
sudo chmod +x /usr/local/bin/docker-compose
```

###### WordPress Installation

Order erstellen:

```
mkdir my_wordpress 
```

Order öffnen

```
cd my_wordpress 
```

Vorgefertigter Command ausführen. Dieser erstellt ein File und fügt den Inhalt ein.

   -    - ```
                sudo echo "version: '3.3'
                
                services:
                   db:
                     image: mysql:5.7
                     volumes:
                       - db_data:/var/lib/mysql
                     restart: always
                     environment:
                       MYSQL_ROOT_PASSWORD: somewordpress
                       MYSQL_DATABASE: wordpress
                       MYSQL_USER: wordpress
                       MYSQL_PASSWORD: wordpress
                
                   wordpress:
                     depends_on:
                
                - db
                  ge: wordpress:latest
                       ports:
                  - "8000:80"
                    tart: always
                         environment:
                           WORDPRESS_DB_HOST: db:3306
                           WORDPRESS_DB_USER: wordpress
                           WORDPRESS_DB_PASSWORD: wordpress
                           WORDPRESS_DB_NAME: wordpress
                    volumes:
                        db_data: {} > docker-compose.yml
            ```

            ​    

Nun führen Sie aus:

```
docker-compose up -d

Das Docker ocker-compose.yml file:
 Der Wordpress Webserver muss auf die Datenbank über einen Port verbinden können. Hier habe ich denn Standard Port 3306 belassen, da dieser ohne Forwarding sowieso nach aussen gezeigt wird. Der Port 80 muss durch ein weiterleitenden Port nach aussen gezeigt wurde, da dieser Port von Host selbst schon benutzt wird. Hier habe ich mich für den Standard Weiterleitungs HTTP Port entschieden: 8080.
```

##### Vorgefertigtes Bash Script um WordPress zu installieren

1)Nehmen Sie das fertige Bashscript von meinem repository.

2)Legen Sie es auf Ihrer Ubuntu Maschine ab

3)Ändern Sie die Berechtigung mit:

```
chmod 775 <Scriptname> 
4) Führen Sie das Script mit ./Scriptname aus
```

Wie ich auf die Wordpressseite zugreife: http://192.168.60.101:8000

### Kapitel 3 Testing

| Was wird getestet                           | Wie                                                       | Erfolgreich? |
| :------------------------------------------ | --------------------------------------------------------- | ------------ |
| Läuft der Container?                        | docker ps                                                 | ja           |
| Komme ich auf die Komme ich Wordpressseite? | http://ip:8000                                            | ja           |
| Funktioniert das Bash Script?               | Script auf die VM kopieren und mit ./Scriptname abspielen | ja           |
| Funktioniert das Vagrantfile                | Vagrantfile mit vagrant up ausführen                      | ja           |
| Sind die Images vorhanden?                  | docker images                                             | ja           |
| Funktioniert das docker compose             | docker-compose up -d                                      | Ja           |

![1554818160674](C:\Users\samue\AppData\Roaming\Typora\typora-user-images\1554818160674.png)

### Kapitel 4

Anfangs dachten wir, dass wir nicht auf unser WordPress Service vom Notebook aus zugreifen können. Wir haben völlig am falschen Ort gesucht. Uns war völlig unklar an was es liegen könnte. Die Antwort war aber extrem simple und zwar haben wir die 127.0.0.1:8000 verwendet. Das funktioniert natürlich nicht... Man muss auf den epn0s8 Adapter zugreifen, damit es möglich ist die Wordpressseite zu öffnen.



##### Wie ich mit Docker ein eigener einfacher Container erstellt habe:

Zuerst habe ich mir ein Dockerfile angelegt. Darin habe ich ganz einfach definiert, dass er die Linux alpine version installieren soll, einfach weil die so mega klein ist. und dann einen Text ausgibt. Das Dockerfile hat so ausgesehen:

```
From alpine
CMD ["echo",  "Hello World, das ist mein eigener Container"]
```



`vagrant@docker:~$ vi Dockerfile`
`vagrant@docker:~$ docker build .`
`Sending build context to Docker daemon  20.99kB`
`Step 1/2 : From alpine`
 `---> cdf98d1859c1`
`Step 2/2 : CMD ["echo",  "Hello World, das ist mein eigener Container"]`
 `---> Running in efe3b4da6902`
`Removing intermediate container efe3b4da6902`
 `---> 11aaa1c3f67c`
`Successfully built 11aaa1c3f67c`
`vagrant@docker:~$ docker run --name test 11aaa1c3f67c`
`Hello World, das ist mein eigener Container`



Das hat sehr gut funktioniert. Weiter habe ich versucht noch den top Command laufen zu lassen. Der Kommand zeigt alle laufenden Prozesse. Dafür habe ich das DockerFile wie folgt angepasst:

```
From alpine
CMD ["echo",  "Hello World, das ist mein eigener Container"]
RUN top
```

`vagrant@docker:~$ vi Dockerfile`
`vagrant@docker:~$ docker build .`
`Sending build context to Docker daemon  20.99kB`
`Step 1/3 : From alpine`
 `---> cdf98d1859c1`
`Step 2/3 : CMD ["echo",  "Hello World, das ist mein eigener Container"]`
 `---> Using cache`
 `---> 11aaa1c3f67c`
`Step 3/3 : RUN top`
 `---> Running in 7e95d2983eaf`
`Mem: 1451644K used, 596312K free, 19228K shrd, 67124K buff, 953088K cached`
`CPU:  21% usr   5% sys   0% nic  73% idle   0% io   0% irq   0% sirq`
`Load average: 0.04 0.01 0.00 2/233 5`