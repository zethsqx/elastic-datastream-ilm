# Extra Lab: Elastic Datastream ILM

## Copy and Paste everything to Dev Tools
```bash
## Check number of nodes
GET _cat/nodes?s=i&v

## Set the Cluster interval to be 10s so that ILM will be immediately reflected
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval": "10s"
  }
}

## Set ILM Policy, Rollover at 1day, Delete 1second after rollover
PUT _ilm/policy/logs-foo-bar-1d-delete
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "1d"
          }
        }
      },
      "delete": {
        "min_age": "1s",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

## Create Index Template and attach ILM to the template
## Match any indices with logs-foo*
PUT _index_template/logs-foo-bar-template
{
  "index_patterns": [
    "logs-foo*"
  ],
  "data_stream": {},
  "template": {
    "settings": {
      "index.lifecycle.name": "logs-foo-bar-1d-delete",
      "index.refresh_interval": "1s",
      "number_of_replicas": 0
    }
  },
  "priority": 500
}

## Check there are no existing indices
GET _cat/indices?s=i

## Bulk index 2 docs into the datastream logs-foo.bar-default
POST _bulk
{ "create": { "_index": "logs-foo.bar-default" } }
{ "@timestamp": "2025-12-01T09:00:00Z", "message": "event 1", "log.level": "INFO" }
{ "create": { "_index": "logs-foo.bar-default" } }
{ "@timestamp": "2025-12-01T09:05:00Z", "message": "event 2", "log.level": "ERROR" }

## Check the datastream is created
GET _cat/indices?s=i
## Check the configuration of the datastream
GET _data_stream/logs-foo.bar-default

## Rollover now to check new datastream is generated
POST logs-foo.bar-default/_rollover
## Check the configuration of the datastream
## Observed 2 datastream generation number
GET _cat/indices?s=i
GET _data_stream/logs-foo.bar-default

## Keep running this to observed the changes
GET logs-foo.bar-default/_ilm/explain

DELETE _data_stream/logs-foo.bar-default
```
