[TOC]

# Index management
## Create Index
```
PUT /<index>
```
## Delete index
```
DELETE /<index>
```
## Get index
```
GET /<index>
```
## Index exists
```
HEAD /<index>
```
## Close index
```
POST /<index>/_close
```
## Open index
```
POST /<index>/_open
```
## Shrink index
```
POST /<index>/_shrink/shrunk-twitter-index
```
## Split index
## Clone index
```
POST /<index>/_clone/cloned-twitter-index
```
## Rollover index
## Freeze index
## Unfreeze index
# Mapping management
## Put mapping
## Get mapping
## Get field mapping
## Type exists
# Alias management
## Add index alias
## Delete index alias
## Get index alias
## Index alias exists
## Update index alias
# Index settings
## Update index settings
## Get index settings
## Analyze
# Index templates
## Put index template
## Delete index template
## Get index template
## Index template exists
# Monitoring
## Index stats
## Index segments
## Index recovery
## Index shard stores
# Status management
## Clear cache
## Refresh
## Flush
## Synced flush
## Force merge