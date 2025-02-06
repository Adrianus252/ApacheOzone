# Docker Installationsanleitung (Übung)
- Erstellen eines lokalen Multi-container clusters (https://ozone.apache.org/docs/edge/start/startfromdockerhub.html)

- Optional zur Strukturierung: 
	- Erstellen eines Ordners z.B. docker_test in dem später die Konfigurations- und Docker Compose Datei landet 
	- Dieser Ordnername bestimmt den übergeordneten Namen der Docker Instanz

- Terminal öffnen
- Download des apache/ozone Docker Image
```shell
docker pull apache/ozone
```
- Download/Extraktion der Dateien:  **docker-compose.yaml** und **docker-config**
```bash
docker run apache/ozone cat docker-compose.yaml > docker-compose.yaml
docker run apache/ozone cat docker-config > docker-config
```
- Starten des Clusters mit docker-compose und 3 DataNodes
```shell
docker-compose up -d --scale datanode=3
```
- Output
```shell
 [+] Running 8/8
 ✔ Network docker_ozone_default        Created
 ✔ Container docker_ozone-scm-1        Started
 ✔ Container docker_ozone-datanode-3   Started
 ✔ Container docker_ozone-recon-1      Started
 ✔ Container docker_ozone-datanode-2   Started
 ✔ Container docker_ozone-datanode-1   Started
 ✔ Container docker_ozone-om-1         Started
 ✔ Container docker_ozone-s3g-1        Started 
```

- ACHTUNG bei Fehler aufgrund des Dateiformat beim Start von Docker müssen **docker-compose.yaml** und **docker-config** in UTF-8 konvertiert werden. 
	- z.B. mit Notepad++ (change Encoding)
	- Alternativ die Dateien aus dem Github Repository (https://github.com/Adrianus252/ApacheOzone) herunterladen

- Öffne Bash (Ozone Manager):
```bash
# z.B. docker exec -it docker_ozone-om-1 bash
```

```bash
docker exec -it <dockername>-om-1 bash
```
### Überprüfung ob Ozone CLI verfügbar ist: 

```bash
ozone sh volume list
```
### Am Ende der Übung kann die Umgebung wieder entfernt werden
```shell
docker-compose down -v
```
- Output
```shell
[+] Running 8/8
 ✔ Container docker_ozone-scm-1       Removed 
 ✔ Container docker_ozone-datanode-1  Removed
 ✔ Container docker_ozone-recon-1     Removed  
 ✔ Container docker_ozone-datanode-3  Removed
 ✔ Container docker_ozone-s3g-1       Removed
 ✔ Container docker_ozone-om-1        Removed
 ✔ Container docker_ozone-datanode-2  Removed
 ✔ Network docker_ozone_default       Removed
```

# Installation Bare Metal 
- https://ozone.apache.org/docs/edge/start/onprem.html
- Benötigte Komponenten: 
	- Ozone Manager(OM)
	- Storage Container Manager(SCM)
	- Datanodes
## 1. Installation und Konfiguration
- Auf allen Maschinen im Cluster: entpacke `ozone<version>'
- Generierung von **ozone-site.xml** mit `ozone genconf <path>`
- Konfiguration von **ozone-site.xml**
- Definition der Speicherorts der Metadaten (schnelle SSD)
	- SCM und OM schreiben auf diesen Pfad.
```XML
<property>
  <name>ozone.metadata.dirs</name>
  <value>/data/disk1/meta</value>
</property>
```
- Definition: SCM Name (weitere Einstellungen wie Port, Adresse)
```xml
<property>
  <name>ozone.scm.names</name>
  <value>scm.hadoop.apache.org</value>
</property>
```
- Definition: OM Adresse
```xml
<property>
   <name>ozone.om.address</name>
   <value>ozonemanager.hadoop.apache.org</value>
</property>
```
- Definition: Speicherort von Datanode ID(unique)
```xml
<property>
  <name>ozone.scm.datanode.id.dir</name>
  <value>/data/disk1/meta/node</value>
</property>
```
- Schließlich muss die Konfigurationsdatei ozone-site.xml nach `<ozone_directory>/etc/hadoop` verteilt/kopiert werden.
## 2. Starten des Clusters
- Initialisierung und Starten von Storage Container Manager
	- Erstellung der Strukturen auf Disks
```shell
ozone scm --init
ozone --daemon start scm
```
- Initialisierung und Starten des Ozone Manager
```shell
ozone om --init
ozone --daemon start om
```
- Starten von Datanodes
```shell
ozone --daemon start datanode
```

