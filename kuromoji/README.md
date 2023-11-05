# kuromoji

## 参考

- [Elasticsearchで日本語検索を扱うためのマッピング定義 (2021/11/05)](https://techblog.zozo.com/entry/elasticsearch-mapping-config-for-japanese-search)
- [Elastic Docs > Elasticsearch Plugins and Integrations [8.10] > Analysis plugins
ICU analysis plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.10/analysis-icu.html)
- [Elastic Docs > Elasticsearch Plugins and Integrations [8.10] > Analysis plugins
Japanese (kuromoji) analysis plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.10/analysis-kuromoji.html)

## 手順

1. `$ docker exec -it kuromoji-es01-1 sh`
2. `$ bin/elasticsearch-plugin install analysis-icu`
3. `$ bin/elasticsearch-plugin install analysis-kuromoji`
4. `$ exit`
5. `$ docker restart kuromoji-es01-1`
    - [Step 3: restart docker container to apply config changes](https://medium.com/@harshvardhan.singh/snapshot-restore-data-for-elasticsearch-on-docker-86c048d5246f)
6. `http://localhost:5601/app/home#/tutorial_directory/fileDataViz`
7. csvを選択 or drag and drop
8. `import > Advanced`
9. `Add combined field > Add geo point field`
    - Latitude field: 緯度
    - Longitude field: 経度
    - Geo point field: coordinates
10. index settings

    ```json
    {
      "number_of_shards": 1,
      "analysis": {
        "analyzer": {
          "kuromoji_normalize": {
            "char_filter":[
              "icu_normalizer"
            ],
            "tokenizer": "kuromoji_tokenizer",
            "filter": [
              "kuromoji_baseform",
              "kuromoji_part_of_speech",
              "cjk_width",
              "ja_stop",
              "kuromoji_stemmer",
              "lowercase"
            ]
          }
        }
      }
    }
    ```

11. Mappings

    ```json
    {
      "properties": {
        "原典資料コード": {
          "type": "long"
        },
        "大字・字・丁目区分コード": {
          "type": "long"
        },
        "大字町丁目コード": {
          "type": "long"
        },
        "大字町丁目名": {
          "type": "text",
          "analyzer": "kuromoji_normalize"
        },
        "市区町村コード": {
          "type": "long"
        },
        "市区町村名": {
          "type": "text",
          "analyzer": "kuromoji_normalize"
        },
        "経度": {
          "type": "double"
        },
        "緯度": {
          "type": "double"
        },
        "都道府県コード": {
          "type": "long"
        },
        "都道府県名": {
          "type": "text",
          "analyzer": "kuromoji_normalize"
        }
      }
    }
    ```

12. `Search > Content > Indices > {作ったindex}`
13. `{作ったindex} > Overview > API > Generate an API key`
    - `T0J4Q25vc0JmNXQwY0JNTHBLaUQ6SUhjMEhISkdUVnU0VTN3ZE9FMFIxQQ==`
14. `Configure your client`

    ```sh
      export ES_URL="https://localhost:9200"
      export API_KEY="T0J4Q25vc0JmNXQwY0JNTHBLaUQ6SUhjMEhISkdUVnU0VTN3ZE9FMFIxQQ=="
    ```

15. このままじゃAPI叩けない

    ```sh
      curl "${ES_URL}/kuromoji" \
      -H "Authorization: ApiKey "${API_KEY}"" \
      -H "Content-Type: application/json"
    ```

16. `$ docker cp kuromoji-es01-1:/usr/share/elasticsearch/config/certs/ca/ca.crt .`
17. やっと叩ける

    ```sh
      sudo curl --cacert ca.crt "${ES_URL}/kuromoji"\
      -H "Authorization: ApiKey "${API_KEY}"" \
      -H "Content-Type: application/json" | jq .
    ```

    ```json
      {
        "kuromoji": {
          "aliases": {},
          "mappings": {
            "_meta": {
              "created_by": "file-data-visualizer"
            },
            "properties": {
              "原典資料コード": {
                "type": "long"
              },
              "大字・字・丁目区分コード": {
                "type": "long"
              },
              "大字町丁目コード": {
                "type": "long"
              },
              "大字町丁目名": {
                "type": "text",
                "analyzer": "kuromoji_normalize"
              },
              "市区町村コード": {
                "type": "long"
              },
              "市区町村名": {
                "type": "text",
                "analyzer": "kuromoji_normalize"
              },
              "経度": {
                "type": "double"
              },
              "緯度": {
                "type": "double"
              },
              "都道府県コード": {
                "type": "long"
              },
              "都道府県名": {
                "type": "text",
                "analyzer": "kuromoji_normalize"
              }
            }
          },
          "settings": {
            "index": {
              "routing": {
                "allocation": {
                  "include": {
                    "_tier_preference": "data_content"
                  }
                }
              },
              "number_of_shards": "1",
              "provided_name": "kuromoji",
              "creation_date": "1699167122858",
              "analysis": {
                "analyzer": {
                  "kuromoji_normalize": {
                    "filter": [
                      "kuromoji_baseform",
                      "kuromoji_part_of_speech",
                      "cjk_width",
                      "ja_stop",
                      "kuromoji_stemmer",
                      "lowercase"
                    ],
                    "char_filter": [
                      "icu_normalizer"
                    ],
                    "tokenizer": "kuromoji_tokenizer"
                  }
                }
              },
              "number_of_replicas": "1",
              "uuid": "LAexR7BaTIOKb4yt2kKjqw",
              "version": {
                "created": "8100499"
              }
            }
          }
        }
      }
    ```

18. analyzerの動作確認

    ```sh
      sudo curl -X POST --cacert ca.crt "${ES_URL}/kuromoji/_analyze"\
      -H "Authorization: ApiKey "${API_KEY}"" \
      -H "Content-Type: application/json" \
      -d '{
            "analyzer": "kuromoji_normalize",
            "text": "山の手一条十三丁目"
          }' | jq .
    ```

    ```sh
      {
        "tokens": [
          {
            "token": "山の手",
            "start_offset": 0,
            "end_offset": 3,
            "type": "word",
            "position": 0
          },
          {
            "token": "一",
            "start_offset": 3,
            "end_offset": 4,
            "type": "word",
            "position": 1
          },
          {
            "token": "条",
            "start_offset": 4,
            "end_offset": 5,
            "type": "word",
            "position": 2
          },
          {
            "token": "十",
            "start_offset": 5,
            "end_offset": 6,
            "type": "word",
            "position": 3
          },
          {
            "token": "三",
            "start_offset": 6,
            "end_offset": 7,
            "type": "word",
            "position": 4
          },
          {
            "token": "丁目",
            "start_offset": 7,
            "end_offset": 9,
            "type": "word",
            "position": 5
          }
        ]
      }
    ```

19. fuzzy query で叩いてみる

    ```sh
      sudo curl --cacert ca.crt "${ES_URL}/kuromoji/_search"\
      -H "Authorization: ApiKey "${API_KEY}"" \
      -H "Content-Type: application/json" \
      -d '{
            "query": {
              "fuzzy": {
                "大字町丁目名": {
                  "value": "朝"
                }
              }
            }
          }' | jq .
    ```
