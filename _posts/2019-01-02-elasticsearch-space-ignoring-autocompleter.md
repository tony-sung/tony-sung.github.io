---
title:  "Space ignoring autocompleter with Elasticsearch"
# categories: tech
tags: Elasticsearch
---

# 상황
Edge NGram Tokenizer를 활용하여 Elasticsearch에서 [자동완성 시스템](https://tony-sung.github.io/tech/elasticsearch-autocompleter/)을 만들었다. 

"고소한 참기름"이라는 TEXT를 인덱싱하였고 이제 "고" 또는 "고소"와 같이 글자의 일부를 입력하여 이 document를 찾을 수 있다.
```
GET autocomplete-v2/_search
{
  "query":{
    "match":{
      "name":"고"
    }
  }
}
```

그렇다면 **"고소한참"이라고 스페이스 없이 붙여쓰면 "고소한 참기름"을 찾을 수 있을까? 아니다.**
이유는 우리의 Analyzer로 인덱싱된 토큰은 "고,고소,고소한,참,참기,참기름"이고 "고소한참"이라는 토큰은 저장되지 않았기 때문이다.

```
{
  "tokens": [
    {
      "token": "고",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "고소",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "고소한",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "참",
      "start_offset": 4,
      "end_offset": 5,
      "type": "word",
      "position": 3
    },
    {
      "token": "참기",
      "start_offset": 4,
      "end_offset": 6,
      "type": "word",
      "position": 4
    },
    {
      "token": "참기름",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 5
    }
  ]
}
```

따라서 우리는 "고소한참, 고소한참기, 고소한참기름"이라는 토큰까지 다 저장할 수 있도록 space를 무시해서 저장해야한다.

# 방법
몇 가지 Trick을 사용했다.
```
PUT autocomplete-v3
{
  "analysis": {
    "filter":{
      "autocomplete_filter": {         // (1)
          "type": "edge_ngram",
          "min_gram": "1",
          "max_gram": "20"
      },
      "word_joiner": {                 // (2)
        "type": "word_delimiter",
        "catenate_all": true
      }
    },
    "analyzer": {
      "autocomplete": {
        "type":"custom",
        "tokenizer": "keyword",        // (3)
        "filter": [
            "lowercase",
            "autocomplete_filter",
            "word_joiner"
        ]
      }
    }
  }
}
```

(1) 우선 edge_ngram tokenizer를 filter로 분리했다. 이유는 우리가 새로 만들 분석기에 여러 가지 토큰필터를 조합하여 우리가 원하는 바를 얻기 위해서이다.

(2) "word_joiner"라는 word_delimiter 토큰필터를 만들었다. 이 필터는 "catenate_all"값에 true를 주어 스페이스를 무시할 수 있도록 한다.

(3) tokenizer는 "keyword" 토크나이저를 사용했다. 이는 아무 토크나이징도 하지 않고 전적으로 토큰필터에 의해서만 토크나이징을 하도록 한다. 명시하지 않으면 "standard" 토크나이저를 사용하므로 주의한다.

이를 통해 우리는 "고소한 참기름"이라는 글자를 다음의 순서로 분석할 수 있다.
1. keyword tokenizer (아무것도 하지 않음): 고소한 참기름
2. word_jointer filter: 고소한, 참기름, 고소한참기름
3. autocomplete filter: 고, 고소, 고소한, 고소한참, 고소한참기, 고소한참기름, 참, 참기, 참기름

우리가 만든 autocomplete 분석기를 테스트해보자.
```
GET autocomplete-v3/_analyze
{
  "analyzer": "autocomplete",
  "text":     "고소한 참기름"
}
```

목표한 바와 같이 다음의 결과를 얻을 수 있다.
```
{
  "tokens": [
    {
      "token": "고",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 0
    },
    {
      "token": "고소",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 0
    },
    {
      "token": "고소한",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 0
    },
    {
      "token": "참",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 3
    },
    {
      "token": "고소한참",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 3
    },
    {
      "token": "참기",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 5
    },
    {
      "token": "고소한참기",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 5
    },
    {
      "token": "고소한참기름",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 6
    },
    {
      "token": "참기름",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 7
    }
  ]
}
```

이제 "고소한참" 또는 "고소한참기" 등과 같이 스페이스를 무시한 검색어를 입력해도 원하는 값을 얻을 수 있다.

```
GET autocomplete-v3/_search
{
  "query":{
    "match":{
      "name":"고소한참"
    }
  }
}
```

# Reference
- [Writing a space-ignoring autocompleter with ElasticSearch](https://medium.com/@davedash/writing-a-space-ignoring-autocompleter-with-elasticsearch-6c3c28e3a974)

- [Word Delimiter Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-word-delimiter-tokenfilter.html)
