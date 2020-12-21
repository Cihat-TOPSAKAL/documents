# Elasticsearch

* Delete index;
  
        curl -X DELETE 'http://localhost:9200/<INDEX_NAME>'

* List All indexes;

        curl -X GET 'http://localhost:9200/_cat/indices?v'

* List All Docs in Index

        curl -X GET 'http://localhost:9200/<INDEX_NAME>/_search'

* Query Using;

        curl -X GET http://localhost:9200/<INDEX_NAME>/_search?q=<Parameter>:<Value>

* Add Data;

        curl -XPUT --header 'Content-Type: application/json'http://localhost:9200/<NODE_NAME>/_doc/1 -d '{"<Parameter>" : "<Value>"}'

* Cluster to Add Master Node;

    * elasticsearch.yml

            node.name: <INDEX_NAME>
            node.master: true
            
* List All Nodes Inspect;

        curl -X GET 'http://localhost:9200/_cat/nodes?v'

* Add Indexes;

        curl -X PUT 'http://localhost:9200/<MY_INDEX_NAME>'
    
  