# solr-getting-started

Verzeichnis für Indexe etc.:

```
export SOLR_HOME=/Users/stefan/tmp/solr/
```

oder 

```
export JAVA_OPTS="$JAVA_OPTS -Dsolr.solr.home=/Users/stefan/tmp/solr"
```

oder

```
solr start -cloud -Dsolr.solr.home=/Users/stefan/tmp/solr/
```

Anschliessend Inhalt von `solr/server/solr` in das gewünschte Index-Root-Verzeichnis kopieren. Falls `solr` mit `-D` gestarte wird, muss der Inhalt bereits vorhanden sein.

solr im Cloud Modus starten:
```
solr start -cloud -V
```




********* todo
```
./server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:9983 -cmd upconfig -confname my_new_config -confdir server/solr/configsets/_default/conf/
```

curl -X GET "http://localhost:8983/solr/admin/configs?action=LIST"

curl -X GET "http://localhost:8983/solr/admin/collections?_=1527535447221&action=CREATE&autoAddReplicas=false&collection.configName=my_new_config&maxShardsPerNode=1&name=my_new_collection&numShards=1&replicationFactor=1&router.name=compositeId&wt=json"



 zip -r - * > myconfigset.zip
curl -X POST --header "Content-Type:application/octet-stream" --data-binary @myconfigset.zip "http://localhost:8983/solr/admin/configs?action=UPLOAD&name=myConfigSetFubar"

curl -X GET "http://localhost:8983/solr/admin/collections?action=CREATE&autoAddReplicas=false&collection.configName=myConfigSetFubar&maxShardsPerNode=1&name=test1&numShards=1&replicationFactor=1&router.name=compositeId&wt=json" --> solrconfig.xml not found

curl -X GET "http://localhost:8983/solr/admin/collections?action=CREATE&autoAddReplicas=false&collection.configName=my_new_config&maxShardsPerNode=1&name=test2&numShards=1&replicationFactor=1&router.name=compositeId&wt=json" --> works
*********


Config Set erstellen/hochladen:

```
./server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:9983 -cmd upconfig -confname addressConfigSet -confdir /Users/stefan/Projekte/solr-getting-started/configsets/address/conf/
```

(oder mit REST-API:
```
zip -r - * > address.zip
curl -X POST --header "Content-Type:application/octet-stream" --data-binary @address.zip "http://localhost:8983/solr/admin/configs?action=UPLOAD&name=addressConfigSet"
```
)

Collection erstellen:
```
curl -X GET "http://localhost:8983/solr/admin/collections?action=CREATE&autoAddReplicas=false&collection.configName=addressConfigSet&maxShardsPerNode=1&name=address&numShards=1&replicationFactor=1&router.name=compositeId&wt=json"
```

(Collection löschen:)
```
curl -X GET "http://localhost:8983/solr/admin/collections?action=DELETE&name=address&wt=xml"
```

(Collection reload)
```
curl -X GET "http://localhost:8983/solr/admin/collections?action=RELOAD&name=address&wt=xml"
```

Disable "Data driven schema functionality":
```
curl -X GET "http://localhost:8983/solr/address/config" -d '{"set-user-property": {"update.autoCreateFields":"false"}}'
```

Schema mit verschiedenen Feldern erstellen:

```
curl -X POST "http://localhost:8983/solr/address/schema?wt=json" -d '{add-field: {stored: "true", indexed: "true", name: "egid", type: "string"}}'
curl -X POST "http://localhost:8983/solr/address/schema?wt=json" -d '{"add-field":{"stored":"true","indexed":"true","name":"full_address","type":"text_general","required":"true"}}'
```

Daten importieren:

```
curl -X POST "http://localhost:8983/solr/address/dataimport?indent=on&wt=json" -d '{command=full-import&verbose=false&clean=true&commit=true&core=address&name=dataimport}'
```

Resultat:

```
Last Update: 20:16:13

Indexing completed. Added/Updated: 102384 documents. Deleted 0 documents. (Duration: 05s)
Requests: 1 , Fetched: 102,384 20,477/s, Skipped: 0 , Processed: 102,384 20,477/s
Started: 2 minutes ago
```

Suchen:

```
curl -X GET "http://localhost:8983/solr/address/select?q=full_address:ob*+AND+full_address:gas*+AND+full_address:Eger*"
curl -X GET "http://localhost:8983/solr/address/select?hl=on&hl.fl=full_address&q=full_address:ob*+AND+full_address:gas*+AND+full_address:Eger*"
```