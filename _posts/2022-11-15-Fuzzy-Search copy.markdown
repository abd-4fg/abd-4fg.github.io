---
layout: post
title: "Algolia, Fuzzy search, elastic search"
date: 2022-11-15 08:00:00 +0530
categories: Algolia, Fuzzy search, elastic search
comments_id: 3
---

[Algolia search 사용하기](https://www.algolia.com/)

[elastic search 사용하기]()

[Fuzzy search 알아보기]()

<br>

검색 기능은 앱의 가장 보편적인 기능이라고 볼 수 있습니다.

google에 문자를 하나 칠 때마다 관련 검색어를 나열해주는 기능도 결국 검색 기능이 될 것입니다.

google 관리자 콘솔에서 network탭을 열어보면 문자의 입력때마다 request를 보내고 있는 것을 확인할 수 있었습니다.

그 중 가장 보편적이고 상용화된 서치 툴을 그 원리와 함께 볼 수 있으면 합니다.

<br>

# Algolia Search

<img src='https://res.cloudinary.com/hilnmyskv/image/upload/v1527077656/Algolia_OG_image_m3xgjb.png' style="border-radius: 10px" >

<br>

알고리아 서치는 AI-powered 서치입니다. 관련성을 AI가 제공하고 속도를 극대화 시켰습니다.

알고리아 서치에는 boolean 값 조회, 결과 분류, 카테고리나 세부 상세 설명을 통환 검색 등등 여러가지 기능이 있습니다.\
https://www.algolia.com/doc/guides/managing-results/refine-results

<br>

### 오타는 어떻게 판별할까

<hr>

Algolia의 오타 판별 알고리즘은 Distance로 결정합니다. 실제로 검색된 문자와의 character를 대조하여 그와 비슷한 문자를 대기열에 올려놓는 방식으로 작동합니다.

예를 들어, hello라는 문자를 검색하려고 했을 때, hello로 정확히 검색한다면 hello는 Distance 0으로 검색이 되지만 hllo는 Distance 0으로 해당되는 문자가 없고, Distance 1로써 hello가 있기 때문에 그에 따른 검색 결과를 도출합니다.

strm과 star는 Distance가 2 이므로 Algolia search에서 제공하는 typo tolerance를 견딜 수 있습니다. 물론 google에 strm이라는 회사가 있으므로 Star가 검색되지는 않습니다.

hyphen\"-", space\" "를 판별하는 기능도 제공합니다.

오타가 자주나는 점을 이용해 그 이름을 사용하여 도메인을 장악한다던가 검색 엔진의 최상위로 올리는 행위도 막을 수 있습니다 `Typosquatting`.

<br>

### **Typosquatting을 막는 방법**

<hr>

오타에 대한 이야기로 시작했지만 Algolia의 Dataset을 보면 데이터의 분류 기술이 얼마나 정교한지 알 수 있습니다.

```
[
  {
    "twitter_handle": "BarackObama",
    "nb_followers": 103500000
  },
  {
    "twitter_handle": "BarakObama",
    "nb_followers": 15800
  }
]
```

Algolia에서 예시로 제공한 검색 dataset입니다.

BarakObama라고 오타를 냈을 때에 팔로워 수에 따라서 검색엔진은 BarackObama를 검색하려고 했다는 것을 알 수 있어야 합니다.

Algolia는 `sort-by attribute feature`라는 기능을 소개합니다.

<br>

### **sort-by attribute feature - replicas(복제품)**

<hr>

Algolia는 특별한 분류 기능이 있습니다. 지수 수준에서부터 정제된 dataset을 먼저 생성합니다. **Pre-sorting at indexing time instead of at query time leads to a considerable performance boost.** 미리 분류하는 것은 성능 향상에 물론 영향이 있겠습니다.

고로 replica란 pre-sorting을 위해서 같은 데이터안에서 복제하여 서로 다른 순서의 랭킹을 만드는 것입니다.

때문에 Barack Obama와 Barak Obama의 랭킹을 replica를 통해 팔로워 수와 같은 유의미한 지표로 분류를 해놓은 상태에서 더 상위 랭킹의 검색 결과를 먼저 보여주는 방식으로 작동하게 됩니다.

<br>

# Elastic Search

<img src='https://miro.medium.com/max/558/1*Co95dG0NmGfL-vGMSBtLWQ.png' style="border-radius: 10px; background-color: 'white'" >

<br>

### 그들의 소개

<hr>

Elastic에 대한 기술적인 표현을 사용한 소개입니다.

Elasticsearch는 텍스트, 숫자, 위치 기반 정보, 정형 및 비정형 데이터 등 모든 유형의 데이터를 위한 무료 검색 및 분석 엔진으로 분산형과 개방형을 특징으로 합니다. Elasticsearch는 Apache Lucene을 기반으로 구축되었으며, Elasticsearch N.V.(현재 명칭 Elastic)가 2010년에 최초로 출시했습니다. 간단한 REST API, 분산형 특징, 속도, 확장성으로 유명한 Elasticsearch는 데이터 수집, 보강, 저장, 분석, 시각화를 위한 무료 개방형 도구 모음인 Elastic Stack의 핵심 구성 요소입니다. 보통 ELK Stack(Elasticsearch, Logstash, Kibana의 머리글자)이라고 하는 Elastic Stack에는 이제 데이터를 Elasticsearch로 전송하기 위한 경량의 다양한 데이터 수집 에이전트인 Beats가 포함되어 있습니다.

<br>

### Elastic의 인덱싱 원리

<hr>

먼저 색인(인덱싱)이 어떻게 일어나는지 알아야겠습니다.

`색인한다`라는 표현은 많은 책에서 맨 뒤 키워드를 중심으로 그 키워드가 몇페이지에 있었는지 적어놓은 찾아보기 페이지와 비슷합니다. 당연하게도, 데이터를 저장하는 과정과는 확연히 다르고 ES에서는 데이터를 저장하는 과정에서 일어나기 때문에 색인한다 라는 표현을 사용합니다.

다양한 소스로부터 원시 데이터가 Elasticsearch로 흘러들어가면, Elasticsearch는 일련의 키와 여러 필드의 값을 JSON형태로 저장하고 해당 필드에 데이터를 연결합니다.

Elasticsearch는 역 인덱스라고 하는 데이터 구조를 사용하는데, 이것은 아주 빠른 풀텍스트 검색을 할 수 있도록 설계된 것입니다. 역 인덱스는 문서에 나타나는 모든 고유한 단어의 목록을 만들고, 각 단어가 발생하는 모든 문서를 식별합니다.

<br>

### 역 인덱싱

<hr>

오라클이나 MySql 같은 관계형 DB에서는 내용을 테이블 구조로 저장을 합니다. 테이블 구조에서 특정 필드 내의 특정 값을 찾을 때에 모든 데이터를 거치면서 확인을 하는 절차를 거칩니다. 데이터가 늘어날수록 검색해야 할 대상이 늘어났고 특정 필드에 접근하기 위해 row안의 모든 내용을 읽어야 했습니다.

데이터를 저장할 때 키워드`Term`를 추출하여 만약 역 인덱스가 잇으면 도큐먼트의 id에 바로 접근이 가능하게 됩니다. 데이터가 늘어났을 때 모든 row를 확인하는 것이 아니라 키워드가 가리키는 id의 배열값이 추가되는 것이기 때문에 훨씬 빠른 속도로 검색이 가능하게 되는 것입니다.

<br>

### Kibana && Logstash

<nr>

Kibana는 Elasticsearch 데이터의 실시간 시각화를 제공하며, UI를 통해 애플리케이션 성능 모니터링(APM), 로그, 인프라 메트릭 데이터에 신속하게 접근할 수 있습니다.

Logstash는 데이터를 집계하고 처리하여 Elasticsearch로 전송하는 데 사용됩니다. Logstash는 서버 사이드 오픈 소스 데이터 처리 파이프라인으로, 사용자는 이를 이용해 다양한 소스에서 동시에 데이터를 수집하고, 이를 강화하고 변환한 다음, Elasticsearch에서 색인되도록 할 수 있습니다.

<br>

### Elastic Stack

데이터 수집, 보강, 저장, 분석, 시각화를 위한 무료 개방형 도구 모음인 Elastic Stack의 핵심 구성 요소입니다. Kibana는 시각화, Logstash는 보강, 분석 등에 사용됩니다. Elastic Stack에는 Kibana, Logstash, Elasticsearch와 더불어 데이터를 Elasticsearch로 전송하기 위한 경량의 다양한 데이터 수집 에이전트인 Beats가 있습니다.
