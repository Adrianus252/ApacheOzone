## 1 Hinweise

```shell
# Zugriff auf ozone cli
ozone fs 

# Beispielsyntax 
ozone sh <object> <operation> <--Parameter> /Pfad/Zu/Datei
ozone sh bucket create --enforcegdpr=true /demo/bucketencr

# Hilfe zu Befehlen
# Objekte:volume, bucket, key, snapshot 
# Operation: delete, create, list, put, info
ozone sh <object> <operation> --help

# Bash Umgebung einer bestimmten Docker Instanz öffnen
docker exec -it <ContainerName> bash
# z.B. docker exec -it docker_ozone-om-1 bash
# z.B. docker exec -it docker_ozone-datanode-2 bash

ozone version
```
# 2 Übungen

## 2.1 Erstellen von Objekten
- Erstellen eines Volume namens `demo`
```shell
ozone sh volume create /demo
```
- Liste von Volumes anzeigen lassen
```shell
ozone sh volume list
```

```shell
[ {
  "metadata" : { },
  "name" : "s3v",
  "admin" : "hadoop",
  "owner" : "hadoop",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "usedNamespace" : 0,
  "creationTime" : "2025-02-05T20:57:54.879Z",
  "modificationTime" : "2025-02-05T20:57:54.879Z",
  "acls" : [ {
    "type" : "USER",
    "name" : "hadoop",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  }, {
    "type" : "GROUP",
    "name" : "hadoop",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  } ],
  "refCount" : 0
}, {
  "metadata" : { },
  "name" : "demo",
  "admin" : "hadoop",
  "owner" : "hadoop",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "usedNamespace" : 0,
  "creationTime" : "2025-02-05T21:14:10.675Z",
  "modificationTime" : "2025-02-05T21:14:10.675Z",
  "acls" : [ {
    "type" : "USER",
    "name" : "hadoop",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  }, {
    "type" : "GROUP",
    "name" : "hadoop",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  } ],
  "refCo
```

- Erstellen eines Buckets:  `bucket1`
```shell
ozone sh bucket create /demo/bucket1
```
- Erstellen und hochladen einer Datei in `/demo/bucket1`
- z.B. hello.text
```shell
echo "Hello Ozone" > hello.txt
```

```shell
ozone sh key put /demo/bucket1/hello.txt hello.txt
```
- Ausgabe der Dateien des Buckets `bucket1`
```shell
ozone sh key list /demo/bucket1
```

```shell
[ {
  "volumeName" : "demo",
  "bucketName" : "bucket1",
  "name" : "hello.txt",
  "dataSize" : 12,
  "creationTime" : "2025-02-05T21:15:18.852Z",
  "modificationTime" : "2025-02-05T21:15:20.105Z",
  "replicationConfig" : {
    "replicationFactor" : "THREE",
    "requiredNodes" : 3,
    "replicationType" : "RATIS"
  },
  "metadata" : { },
  "file"
```
## 2.2 Dateien herunterladen
- Herunterladen der Datei mit dem Operator `get`
```shell
ozone sh key get /demo/bucket1/hello.txt downloaded.txt
```

```shell 
cat downloaded.txt 
```
## 2.3 GDPR Compliant Buckets
- Erstelle ein Bucket `bucketencr` im Volume `demo`, welches Keys nur verschlüsselt abspeichert
```shell
ozone sh bucket create --enforcegdpr=true /demo/bucketencr
```
- Erstelle in diesem Bucket eine Datei (Key)
```shell
echo "Verschluesselte Datei" > encr_file.txt
```

```shell
ozone sh key put /demo/bucketencr/encr_file.txt encr_file.txt
```

```shell
ozone sh key list /demo/bucketencr
```

```shell
[ {
  "volumeName" : "demo",
  "bucketName" : "bucketencr",
  "name" : "encr_file.txt",
  "dataSize" : 32,
  "creationTime" : "2025-02-05T21:17:06.946Z",
  "modificationTime" : "2025-02-05T21:17:08.188Z",
  "replicationConfig" : {
    "replicationFactor" : "THREE",
    "requiredNodes" : 3,
    "replicationType" : "RATIS"
  },
  "metadata" : { },
  "file" : true
} ]
```

- Mit dem `info` Operator  können Informationen über Objekte wie Buckets, Keys oder Volumes ausgegeben werden
```shell
ozone sh key info /demo/bucketencr/encr_file.txt
```

```shell
ozone sh bucket info /demo/bucketencr
```

```shell
ozone sh volume info /demo
```
- Dabei wird die Stelle gesucht, bei der die GDPR Verschlüsselung im Objekt markiert ist. 
```shell
{
  "volumeName" : "demo",
  "bucketName" : "bucketencr",
  "name" : "encr_file.txt",
  "dataSize" : 32,
  "creationTime" : "2025-02-05T21:17:06.946Z",
  "modificationTime" : "2025-02-05T21:17:08.188Z",
  "replicationConfig" : {
    "replicationFactor" : "THREE",
    "requiredNodes" : 3,
    "replicationType" : "RATIS"
  },
  "metadata" : {
    "gdprEnabled" : "true"
  },
  "ozoneKeyLocations" : [ {
    "containerID" : 2,
    "localID" : 115816896921600002,
    "length" : 32,
    "offset" : 0,
    "keyOffset" : 0
  } ],
  "file" : true
}
```

```shell
{
  "metadata" : {
    "gdprEnabled" : "true"
  },
  "volumeName" : "demo",
  "name" : "bucketencr",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 96,
  "usedNamespace" : 1,
  "creationTime" : "2025-02-05T21:16:33.756Z",
  "modificationTime" : "2025-02-05T21:16:33.756Z",
  "sourcePathExist" : true,
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "FILE_SYSTEM_OPTIMIZED",
  "owner" : "hadoop",
  "link" : false
}
```

- Lade die verschlüsselte Datei aus dem Bucket `bucketencr` Datei herunter. Die Datei wird dabei wieder entschlüsselt.
```shell
ozone sh key get /demo/bucketencr/encr_file.txt downloaded_decr.txt
```

```shell
cat downloaded_decr.txt
```
## 2.4 Inspektion von Datanodes
- Schaue, dir die Datanodes auf der Command Line an
```shell
ozone admin datanode list
```

```shell
Datanode: c404f568-ef38-4fd2-b921-c77e8bc56a84 (/default-rack/172.18.0.8/docker_ozone-datanode-2.docker_ozone_default/3 pipelines)
Operational State: IN_SERVICE
Health State: HEALTHY
Related pipelines:
7d020f25-52ed-4a91-8926-730a0009ede9/RATIS/THREE/RATIS/OPEN/Follower
4c901574-8279-44a5-b56c-88d1e29b9849/RATIS/THREE/RATIS/OPEN/Leader
9728179c-561b-4dc8-bd9a-b0be6fbfbd84/RATIS/ONE/RATIS/OPEN/Leader

Datanode: d8a08097-a3ae-49c3-a049-c71f296b8e50 (/default-rack/172.18.0.7/docker_ozone-datanode-3.docker_ozone_default/3 pipelines)
Operational State: IN_SERVICE
Health State: HEALTHY
Related pipelines:
92004a07-bf68-4095-80e6-a90f1cc68b52/RATIS/ONE/RATIS/OPEN/Leader
7d020f25-52ed-4a91-8926-730a0009ede9/RATIS/THREE/RATIS/OPEN/Leader
4c901574-8279-44a5-b56c-88d1e29b9849/RATIS/THREE/RATIS/OPEN/Follower

Datanode: 34c93f7d-78d0-4e71-80a9-6b1cdf4e1a61 (/default-rack/172.18.0.2/docker_ozone-datanode-1.docker_ozone_default/3 pipelines)
Operational State: IN_SERVICE
Health State: HEALTHY
Related pipelines:
7d020f25-52ed-4a91-8926-730a0009ede9/RATIS/THREE/RATIS/OPEN/Follower
4c901574-8279-44a5-b56c-88d1e29b9849/RATIS/THREE/RATIS/OPEN/Follower
f329fb2c-2513-4e17-92c6-6077f06ee657/RATIS/ONE/RATIS/OPEN/Leader
```
- Öffne Bash auf der Instanz eines Datanodes
	- Öffne dazu ein neues Terminal und verbinde dich via Bash mit einem Datanode:
```shell
docker exec -it <Containername> bash
# z.B. docker exec -it docker_ozone-datanode-2 bash
```
 - oder gehe auf einen Datanode in Docker und wähle "Open in terminal"

- Schau dir unter `/data/hdds` die Blöcken sowie Metadata an
- Finde die physischen Blöcke der verschlüsselten und entschlüsselten Dateien
```shell
cat /data/hdds.../chunks
cat /data/hdds.../netadata
ls /data/hdds....
```

```shell
cat /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/1/chunks/115816896921600001.block
# Hello Ozone
cat /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/2/chunks/115816896921600002.block
#G.�&�����z��Bږ^yI��)4����

cat /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/1/metadata/1.container

!<KeyValueContainerData>
checksum: ee5a395709419205cf13262fceea5047b42e06df7b33f7cb27a6e29bf7e3cfae
chunksPath: /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/1/chunks
containerDBType: RocksDB
containerID: 1
containerType: KeyValueContainer
layOutVersion: 2
maxSize: 5368709120
metadata: {}
metadataPath: /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/1/metadata
originNodeId: c404f568-ef38-4fd2-b921-c77e8bc56a84
originPipelineId: 4c901574-8279-44a5-b56c-88d1e29b9849
schemaVersion: '3'
state: OPEN

cat /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/2/metadata/2.container

!<KeyValueContainerData>
checksum: 39c9991301dd4b1fa2e4019772f0765061c81b61befd11985f1d8dd9b5c41447
chunksPath: /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/2/chunks
containerDBType: RocksDB
containerID: 2
containerType: KeyValueContainer
layOutVersion: 2
maxSize: 5368709120
metadata: {}
metadataPath: /data/hdds/hdds/CID-8308d28b-f974-41d7-a5aa-ad6bd51384e9/current/containerDir0/2/metadata
originNodeId: c404f568-ef38-4fd2-b921-c77e8bc56a84
originPipelineId: 7d020f25-52ed-4a91-8926-730a0009ede9
schemaVersion: '3'
state: OPEN
```
## 2.5 Ozone Snapshots
-  Lade eine Datei in `/demo/bucket1`
```shell
echo "Snapshot 1" > snapshot1.txt
```

```shell
ozone sh key put /demo/bucket1/snapshot1.txt snapshot1.txt
```
- Erstelle einen Snapshot des Buckets  `/demo/bucket1`
```shell
ozone sh snapshot create /demo/bucket1
```
- Lade eine zweite Datei hoch 
```shell
echo "Snapshot 2" > snapshot2.txt
```

```shell
ozone sh key put /demo/bucket1/snapshot2.txt snapshot2.txt
```
- Erstelle einen zweiten Snapshot von Bucket in `/demo/bucket1`
```shell
ozone sh snapshot create /demo/bucket1
```
- Anzeigen einer Liste aller Snapshots in Bucket `/demo/bucket1` mit `snapshot` + `list` Operator
```shell
ozone sh snapshot list /demo/bucket1
```

```shell
[ {
  "volumeName" : "demo",
  "bucketName" : "bucket1",
  "name" : "s20250205-212329.265",
  "creationTime" : 1738790609265,
  "snapshotStatus" : "SNAPSHOT_ACTIVE",
  "snapshotId" : "dedcdc55-d6fa-483d-a38e-23fb10b0638e",
  "snapshotPath" : "demo/bucket1",
  "checkpointDir" : "-dedcdc55-d6fa-483d-a38e-23fb10b0638e",
  "referencedSize" : 69,
  "referencedReplicatedSize" : 69,
  "exclusiveSize" : 0,
  "exclusiveReplicatedSize" : 0
}, {
  "volumeName" : "demo",
  "bucketName" : "bucket1",
  "name" : "s20250205-212504.358",
  "creationTime" : 1738790704358,
  "snapshotStatus" : "SNAPSHOT_ACTIVE",
  "snapshotId" : "c9f3c324-fa73-40a0-8c76-901fa1dfcd90",
  "snapshotPath" : "demo/bucket1",
  "checkpointDir" : "-c9f3c324-fa73-40a0-8c76-901fa1dfcd90",
  "referencedSize" : 102,
  "referencedReplicatedSize" : 102,
  "exclusiveSize" : 0,
  "exclusiveReplicatedSize" : 0
} ]
```
- Zeige die Unterschiede zwischen zwei Snapshots mit `snapshot diff` an
```shell
ozone sh snapshot diff /demo/bucket1 <NameSnapshot1> <NameSnapshot2>
```

```shell
Difference between snapshot: s20250205-212329.265 and snapshot: s20250205-212504.358
+       ./snapshot2.txt
```
- Anzeige der Infos eines Snapshot
```shell
ozone sh snapshot info /demo/bucket1 s20250205-030511.101
```

```shell
{
  "volumeName" : "demo",
  "bucketName" : "bucket1",
  "name" : "s20250205-212504.358",
  "creationTime" : 1738790704358,
  "snapshotStatus" : "SNAPSHOT_ACTIVE",
  "snapshotId" : "c9f3c324-fa73-40a0-8c76-901fa1dfcd90",
  "snapshotPath" : "demo/bucket1",
  "checkpointDir" : "-c9f3c324-fa73-40a0-8c76-901fa1dfcd90",
  "referencedSize" : 102,
  "referencedReplicatedSize" : 102,
  "exclusiveSize" : 0,
  "exclusiveReplicatedSize" : 0
}
```
## 2.6 Quota
-  Erstelle ein neues volume `demoquo` mit der Einschränkung bezüglich der Namen (Maximum 2)
```shell
ozone sh volume create --namespace-quota 2 /demoqu
```
- Versuche 3 Buckets in diesem Volume zu erstellen
```shell
ozone sh bucket create /demoqu/bucket1
ozone sh bucket create /demoqu/bucket2
ozone sh bucket create /demoqu/bucket3
```
- Beim Erstellen wird ein Fehler auftreten, da nur 2 Buckets in diesem Volume erlaubt sind 
```shell
QUOTA_EXCEEDED The namespace quota of Volume:demoqu exceeded: quotaInNamespace: 2 but namespace consumed: 3.
```
- Kontrolliere die tatsächlich und die erlaubte Anzahl der Namespaces für das Volume
```shell
ozone sh volume info /demoqu
```

```shell
{
  "metadata" : { },
  "name" : "demoqu",
  "admin" : "hadoop",
  "owner" : "hadoop",
  "quotaInBytes" : -1,
  "quotaInNamespace" : 2,
  "usedNamespace" : 2,
  "creationTime" : "2025-02-05T21:27:04.509Z",
  "modificationTime" : "2025-02-05T21:27:04.509Z",
  "acls" : [ {
    "type" : "USER",
    "name" : "hadoop",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  }, {
    "type" : "GROUP",
    "name" : "hadoop",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  } ],
  "refCount" : 0
}
```
- erhöhe Anzahl Buckets im Volume auf 5 mit `setquota`
- danach kann ein neues Bucket ohne Fehlermeldung erstellt werden
```shell
bin/ozone sh volume setquota --namespace-quota 10 /demoqu
```
- Versuche jetzt ein drittes Bucket zu erstellen
```shell
ozone sh bucket create /demoqu/bucket3
```
## 2.7 Ozone Insight
## Use Ozone Insight
- Zeige Komponenten an
```shell
ozone insight list
```

```shell
scm.node-manager                     SCM Datanode management related information.
scm.replica-manager                  SCM closed container replication manager
scm.event-queue                      Information about the internal async event delivery
scm.protocol.block-location          SCM Block location protocol endpoint
scm.protocol.heartbeat               SCM Datanode protocol endpoint
scm.protocol.container-location      SCM Container location protocol endpoint
scm.protocol.security                SCM Block location protocol endpoint
om.key-manager                       OM Key Manager
om.protocol.client                   Ozone Manager RPC endpoint
datanode.pipeline                    More information about one ratis datanode ring.
datanode.dispatcher                  Datanode request dispatcher (after Ratis replication)
```
- Benutze `ozone insight metrics` um dir Metriken zu verschiedenen Komponenten anzuschauen (z.B. Heartbeat)
```shell
ozone insight metrics scm.protocol.block-location
```

```shell
Metrics for `scm.protocol.block-location` (SCM Block location protocol endpoint)
RPC connections
  Open connections: 0
  Dropped connections: 0
  Received bytes: 5261
  Sent bytes: 44652
RPC queue
  RPC average queue time: 0.0
  RPC call queue length: 0
RPC performance
  RPC processing time average: 1.0
  Number of slow calls: 0
Message type counters
  Number of AllocateScmBlock calls: 4
  Number of DeleteScmKeyBlocks calls: 0
  Number of GetScmInfo calls: 2
  Number of SortDatanodes calls: 3
  Number of AddScm calls: 0
```
- Stelle eine Verbindung mit `ozone insight logs` zum Client Protokoll Service her 
```shell
ozone insight logs om.protocol.client 
```
- In einem anderen Terminal sollen jetzt Daten an ein beliebiges Bucket gesendet werden. 
```shell
docker exec -it docker_ozone-datanode-2 bash
```

```shell
ozone sh volume create /demo
```

```shell
echo "Hello Ozone" > hello.txt
```

```shell
ozone sh key put /demo/bucket1/hello.txt hello.txt
```
- Im ersten Terminal können jetzt die Ereignisse verfolgt werden
```shell
[OM] 2025-02-05 21:33:19,367 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol ServiceList request is received
[OM] 2025-02-05 21:33:19,556 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol CreateVolume request is received
[OM] 2025-02-05 21:33:27,346 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol ServiceList request is received
[OM] 2025-02-05 21:33:27,525 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol InfoVolume request is received
[OM] 2025-02-05 21:33:27,539 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol InfoBucket request is received
[OM] 2025-02-05 21:33:27,577 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol CreateKey request is received
[OM] 2025-02-05 21:33:28,818 [DEBUG|org.apache.hadoop.ozone.protocolPB.OzoneManagerProtocolServerSideTranslatorPB|OzoneProtocolMessageDispatcher] OzoneProtocol CommitKey request is received
```


























