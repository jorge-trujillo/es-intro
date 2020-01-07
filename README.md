# Elasticsearch Toy Example

## Running ES

The simplest way to run ES is using Docker. To run version `7.5.1` using docker:

```bash
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.5.1
```

You can also use the provided `compose` file, which will also has a volume mount to store the data locally. This means your test data will _survive container restarts_.

```
docker-compose -f es-compose.yml up -d
```

Finally, be sure to install the Chrome ES Head extension to see your server:
https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm?hl=en-US

## Schema and Data

ES indices are bootstrapped using a JSON mapping, which contains all the field definitions for the index. ES can attempt to "infer" field types if you do not provide field definitions, but in most cases this is not recommended. This toy example contains the following:

* **default_schema.json** - Sample mappings for a `persons` index, containing one type for `person`
* **sample_data.json** - Sample bulk data containing 3 records

The following steps assume you are running a local instance of ES on your machine on port 9200.

To create the index:

```bash
curl -XPUT -H "Content-Type: application/json" 'http://localhost:9200/_template/default_schema' -d @default_schema.json
```

To index records:

```bash
curl -XPOST -H "Content-Type: application/json" 'http://localhost:9200/_bulk' --data-binary @sample_data.json
```

You can use Kopf to issue queries to index, or you can use curl. Here is an example index, which just leverages the name field:

### Get records that match "grey" or "bar"

```json
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "name_t": {
              "query": "grey"
            }
          }
        },
        {
          "match": {
            "name_t": {
              "query": "bar"
            }
          }
        }
      ]
    }
  }
}
```

### Filter persons first that worked more than 10 years, and then find records that match a grade 20

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "start_dt": {
              "lte": "now-10y/d"
            }
          }
        },
        {
          "term": {
            "grade_i": 20
          }
        }
      ]
    }
  }
}
```

Of course, this is a very simple example and vastly more complex queries are possible.
See the excellent ES docs for more: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html
