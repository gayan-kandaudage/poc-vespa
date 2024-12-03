True  reliable and scalable AI seach engine https://vespa.ai/ with features https://vespa.ai/features/#vector-text-and-structured-search. 

Documentation https://search.vespa.ai/

Open source project https://github.com/vespa-engine/vespa/ with Apache 2.0 license

Architecture https://docs.vespa.ai/en/overview.html, https://docs.vespa.ai/en/elasticity.html, https://docs.vespa.ai/en/operations-selfhosted/multinode-systems.html

## Clone the repo and navigate to root of the project

https://bitbucket.org/rambase/poc-for-search-engine-unifiedseachengine/src/main/vespa/

## 1 Pull image

docker pull vespaengine/vespa

## 2 Deploy Vespa application package

Make sure vespa\app folder has data before mouting otherwise it will complain for empty content
docker run --detach --name vespa --hostname vespa-container --publish 8080:8080 --publish 19071:19071 --volume C:\Users\17604\Personal\KT\Hatteland\SearchEngine\Poc\poc-for-search-engine-unifiedseachengine\vespa\app:/app vespaengine/vespa

## 3 Update configurations and upload from container

docker exec -it vespa bash
/opt/vespa/bin/vespa-deploy prepare /app
/opt/vespa/bin/vespa-deploy activate

## 4 Verification commands

### vespa container loggsdocker logs vespa

tail -f /opt/vespa/logs/vespa/vespa.log
/opt/vespa/bin/vespa-deploy status

## 6 Monitoring

Ref https://docs.vespa.ai/en/operations-selfhosted/monitoring.html, https://search.vespa.ai/search?query=How%20to%20set%20up%20monitoring%3F, https://cloud.vespa.ai/en/monitoring, https://docs.vespa.ai/en/operations-selfhosted/monitoring.html

http://localhost:19071/logs

http://localhost:19071/state/v1/health

http://localhost:19071/ApplicationStatus

## 7 Open postman colection to understand and test vespa

you can use postman collection on roof folder to test Vaspa.postman_collection.xml

## 8 Ingestion data

curl --header "Content-Type:application/json" --request POST --data-binary '{
  "put": "id:searchcontent:searchcontent::1",
  "fields": {
    "id": 1,
    "title": "First Document",
    "body": "This is the content of the first document."
  }
}' http://localhost:8080/document/v1/searchcontent/searchcontent/docid/1

## 9 Search

Ref https://docs.vespa.ai/en/query-language.html, https://docs.vespa.ai/en/text-matching.html
curl "http://localhost:8080/search/?query=title:first"

### Multi-phase Ranking with Business Logic

https://docs.vespa.ai/en/ranking.html

Combination of Structured and Unstructured Data Queries
curl "http://localhost:8080/search/?yql=select * from sources * where (title contains 'first') and title contains 'document'"

### Machine Learning

ML models directly in the ranking process.

# 10 Metrics and Prometheus

https://docs.vespa.ai/en/reference/vespa-set-metrics-reference.html

http://localhost:19071/state/v1

http://localhost:19071/metrics/v2

http://localhost:19071/prometheus/v1

# 11 ingestion of QMS Data for role permission, multi tenese support

Sample-data folder has prepopulated 2 wiki pages, testing can be achived with https://bitbucket.org/rambase/poc-for-search-engine-unifiedseachengine/src/main/vespa/Vespa%20QMS.postman_collection.json

## Create of JSON payload wiki pages (high volume data load)

printf '{
  "fields": {
    "title": "ISO 9001:2015 Quality Management System",
    "description": "Guidelines for establishing, implementing, maintaining, and improving a quality management system.",
    "content": "%s",
    "content_snippet": "ISO-9001:2015 QMS with focus on customer requirements and satisfaction.",
    "content_type": "Quality Management Standard",
    "version": 2015,
    "author": {
      "id": "author456",
      "firstName": "Jane",
      "lastName": "Doe"
    },
    "verifier": {
      "id": "verifier789",
      "firstName": "John",
      "lastName": "Smith"
    },
    "approver": {
      "id": "approver101",
      "firstName": "Alice",
      "lastName": "Johnson"
    },
    "permissions_group_ids": {
      "quality_manager": "view",
      "admin": "manage",
      "compliance_officer": "edit"
    }
  }
}' "$HTML_CONTENT" > payload.json

HTML_CONTENT=$(<document2.html sed 's/\\/\\\\/g; s/"/\\"/g; s/\n/ /g')

## curl ingestion of high volume data

curl -X POST "http://localhost:8080/document/v1/searchcontent/searchcontent/docid/ISO-37001" \
-H "Content-Type: application/json" \
-d @payload2.json

## search of ingested data filter for access control

### unrestricted

curl -X POST "http://localhost:8080/search/" -H "Content-Type: application/json" -d '{
  "yql": "select * from sources * where title contains \"ISO\" and permissions_group_ids contains {\"view\": \"role_all\"}",
  "ranking": "default",
  "hits": 10
}'

### restricted_role

curl -X POST "http://localhost:8080/search/" -H "Content-Type: application/json" -d '{
  "yql": "select * from sources * where title contains \"ISO\" and permissions_group_ids contains {\"view\": \"restricted_role\"}",
  "ranking": "default",
  "hits": 10
}'