1.删除索引

```txt
DELETE /finish_collection_data
```



2.创建索引

```txt
PUT finish_collection_data
```



3.配置mapping

```txt
POST /finish_collection_data/_mapping/
{
  "properties": {
    "contentType": {
      "type": "long"
    },
    "curPageIndex": {
      "type": "long"
    },
    "enableInterception": {
      "type": "boolean"
    },
    "enableLinkedCollection": {
      "type": "boolean"
    },
    "enableStatic": {
      "type": "boolean"
    },
    "endTime": {
      "type": "long"
    },
    "existPostFormBody": {
      "type": "boolean"
    },
    "flag": {
      "type": "long"
    },
    "parserFullName": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "post": {
      "type": "boolean"
    },
    "postFormBody": {
      "type": "object"
    },
    "preId": {
      "type": "long"
    },
    "priority": {
      "type": "long"
    },
    "proxyIp": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "proxyPassword": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "proxyPort": {
      "type": "long"
    },
    "proxyUsername": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "referer": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "reqCookies": {
      "type": "object"
    },
    "reqHeads": {
      "type": "object"
    },
    "reqParams": {
      "type": "object"
    },
    "requestMediaType": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "savePosition": {
      "type": "boolean"
    },
    "select": {
      "properties": {
        "id": {
          "type": "long"
        }
      }
    },
    "startTime": {
      "type": "long"
    },
    "test": {
      "type": "boolean"
    },
    "text": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "timeout": {
      "type": "long"
    },
    "trust": {
      "type": "boolean"
    },
    "type": {
      "type": "long"
    },
    "url": {
      "type": "keyword"
    },
    "useProxy": {
      "type": "boolean"
    },
    "useProxyAuth": {
      "type": "boolean"
    },
    "used": {
      "type": "boolean"
    },
    "userAgent": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "validateHttps": {
      "type": "boolean"
    },
    "wsEndPoint": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "collectionNodeIp": {
          "type": "keyword"
    },
    "collectionSuccessful": {
          "type": "keyword"
        }
  }
}

```

