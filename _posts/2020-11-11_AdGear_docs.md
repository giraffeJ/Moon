---
layout: post
title:  "AdGear Docs"
date:   2020-11-11
excerpt: "-"
tag:
comments: false
---

<!-- toc:start -->
Table of Contents
* [acr-linear-ads.md](#acr-linear-ads.md)
* [auction.md](#auction.md)
* [boolean.md](#boolean.md)
<!-- toc:end -->

# acr-linear-ads.md <a id="acr-linear-ads.md"></a>

## Context
: Linear Ads within the ACR context is the name giving the ads shown on "linear" TV. In other words, ads shown on a ye olde TV channels as opposed to streaming services.
ACR can therefore detect and provide an event stream that will tell us whenever a TV is watching a given ad on their TV. This data is currently ingested into CDW for analytical use cases and also to build audiences within Envision and the Audience Planner.
## Frequency Capping
: 한 명의 사람에게 동일한 내용의 광고를 일정 수 이상 반복하여 보여주지 않는 것.
+ linear ads에 대한 use case는 frequency capping에 활용될 수 있다.
```
ACR = Automatic content recognition
: Identification technology to recognize content played on a media device or present in a media file.
동영상 광고 유형
1. Linear Ads = 선형 광고. ex. 유튜브 시청 중간에 삽입되어 영상의 흐름을 끊고 진행되는 광고 등.
2. Non-Linear Ads = 비선형 광고. 동영상 재생과 동시에 그 위에 배너 형식 등으로 표현되는 광고.
3. Companion Ads = 동영상이나 메인 컨텐츠를 가리지 않고 별개로 진행되는 광고. ex. 사이드 배너 광고.
```

------

# auction.md <a id="auction.md"></a>

## Business Purpose
+ impression 하나 당 bid 하나만을 exchange에 회신한다
+ deal들을 위한 account들이 필요하다
+ O&O는 우선순위에 대한 특별한 로직을 가지고 있다.
+ dsp는 대부분 first-price auction이다
+ ties는 공정성을 위해 무작위로 broken되어야 한다.
+ ssp의 auction type은 first/second price 중 하나이다.

## Implementation
+ impression_response object가 input으로 사용된다
  - impression_response는 bidder들로부터의 bid이다.
  - bid price, imp, deal을 포함하고 있다.
+ bid response를 만들기 전 마지막에 시행되는 프로세스이다.

### DSP
+ 대부분 first price auction logic에 적용된다
+ flights간의 tie는 공정을 기하기 위해 randomly broken 된다. (입찰할 flight를 임의로 하나 pick한다?)
+ deal들은 개별적으로 평가된다
  - single bid를 만들기 위해, deal별로 auction을 진행한다
  - deal과 open-market 각각 하나의 winning bid를 유지한다. (we keep one winning bid per deal and one for open-market)
  - 모든 bid들은 exchange로 return 된다.

### O&O
+ 독점가로 거래하고 있기 때문에, bid price는 중요하지 않다
+ priorities가 더 중요하다
+ 그래도 ties broken randomly.

#### Product Priorities
+ deals 사이에서 시행된다
+ O&O priorities별로 하나의 deal을 가진다
+ deal에 연계된 product는 priority를 가진다
+ deal간의 경쟁에서 product priority가 높은 bid가 이긴다.

#### Carousel Ads
+ 여러개의 impression들로 구성
+ 하나의 campaign은 이들 중 하나의 impression만 win할 수 있음
+ ad-podding과 비슷
+ 하나의 bid에서 win하면 다른 impression들에 대한 impression_response를 제거
+ 특정 creative에서만 작동

### SSP
+ auction type 조정 가능 (1st, 2nd)

```
O&O = ?
deal = publisher와 advertiser간의 약속. publisher가 exchange에 요청하여 생성하며, ad request를 보낼 때마다 deal을 지정해주어야 함.
Carousel ad = 여러개의 사진이나 영상으로 구성된 광고. facebook이나 instagram 광고가 대표적
Ad-Podding = 하나의 ad-request로 여러개의 광고를 입찰할 수 있게 함. 광고 중복 등을 막을 수 있음.
```

------

# boolean.md <a id="boolean.md"></a>

## business purpose
+ filtering logic 적용을 위해 일반적인 DSL을 제공한다.

### Product Filters
+ Product는 seller의 app/site의 loose bundle이다. (loost bundle=?)
+ marketplace나 deal에서 이 bundle이 하나의 단위가 된다.
+ SSP로부터 오는 request들에만 적용된다.
+ product를 bid_request에 매칭시키는 데 사용된다.
+ extension을 통해 product의 deal을 bid request에 매칭할 수 있다.
+ SSP bid request의 ACL로서 작용한다. (SSP 기준에 적합하지 않은 product를 걸러낼 수 있어서?)
  - 어떤 buyer가 이 request를 볼 수 있는지를 정한다.
+ 제품 수준에서 지정할 수 있는 간단한 타게팅 룰 선택
+ sell side id에 의존적

### Flight filters
+ flight의 타게팅 요건(=flight가 보고자 하는 inventory)을 표현
+ 모든 flight는 모든 bid_request에 대해 evaluate 된다.
+ 이 프로세스의 결과물(output)은 적합한 flight들과 creative들의 리스트이다.
+ targetting / inventory selection / creative selection을 포함한다.

### Forecasting
+ Flight filters와 동일한 표현
+ 다음 캠페인을 위한 inventory의 availability를 예측
+ flight 구성을 평가하는 데에 중요한 역할
[detail](#forecasting.md)

### Universal Segments
+ [Adgear segments](segments.md)의 dynamic version
+ 

```
DSL = Domain Specific Language?
ACL = Access Control List?
------

# viewing-the-data.md

