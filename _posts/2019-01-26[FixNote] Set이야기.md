---
layout: post
title: "[FixNote] Set 이용하여 중복 제거"
description: "지도 위에 POI 마커 관리를 위해 Swift Set 자료구조 사용하기"
categories: FixNote
tags: [programming, swift, Set, 자료구조,]
redirect_from:
   - /2019/01/19
---

# At a Glance

서버로 부터 받아온 데이터들을 기준으로 마커를 노출 할때 POI ID 중복을 해결해보자.

# Should Be
서버에서 받아온 검색 리스트 데이터를 기반으로 하여 마커 데이터를 다시 구성하고, 이 데이터를 이용하여 지도에 마커를 찍어야 한다.

# AS-IS
마커를 찍기 위한 리스트는 두 종류이고 두 리스트의 성격의 차이는 다음과 같다.

W 리스트. 마커를 찍는다. id는 유니크 하며, 두 개 이상이 존재할 수 없다.
C 리스트. 마커를 찍는다. 한 마커의 id는 유니크 하지만, 하나의 id에 다른 내용을 추가하면 리스트엔 동일 id를 가지고 있는 다른 내용이 나타난다.

# Problem
기존 로직은 cell by cell 로 데이터를 추출하여 마커를 찍기 위한 데이터를 구성하는 로직으로 W 리스트엔 적합하다.
하지만 POI ID가 중복되어 내려오는 데이터가 있는 C리스트는 구분값이 중복되기 때문에 마커가 중복해서 찍히는 문제가 발생한다.

예를 들어 강남역이 12345 라고 하는 POI ID를 가지고 있다고 가정하자. 이때 강남역 11번 출구, 12번 출구 등 역시 12345 의 ID 값을 가지고 있다. 내려오는 데이터가 출구까지 세분화해서 보여주는 검색 리스트라 한다면, 이를 구분하여 마커를 찍었을때 출구 갯수만큼 마커가 찍히게 된다.

# Solution
처음 구상은 Dictionary를 사용하는 것이였다.
Hash의 키 값으로 각 리스트의 ID를 별도로 추출하고, value로 중복 flag를 통해서 구분하려고 했다.

*이 얼마나 비효율적인가..*

**첫 번째로, 로직이 너무 조잡하다.**
별도의 Hash 생성을 하고, 이를 담고, 비교하고 삭제하고 하는 로직이 또 들어가야 하며 중간에 side effect가 날 가능성이 농후하다.

**두 번째로, 리스트의 확장에 취약하다. **
리스트의 데이터를 기반으로 마커를 찍는 것이 맞지만 API는 따로 잡아오고 있다.
그리고 API 특성상 리스트트 offset기반으로 페이징이 되지만, 마커는 한번에 받아와서 뿌려준다.
*(C리스트의 특징...독특)*

**따라서 마커데이터만을 중복 제거 하기 위해 Set을 사용한다**

> Sets
A set stores distinct values of the same type in a collection with no defined ordering. You can use a set instead of an array when the order of items is not important, or when you need to ensure that an item only appears once.
The Swift Programming Language(Apple. Swift 4.1 Edition)

자료구조 Set에 관련 된 내용은 apple 공식 저서인 The Swift Programming Language에 기재되어 있다.
내용을 확인해면 hashvalue를 기준으로 고유한 element를 구성하며, 따라서 Hashable 프로토콜을 따른다.
Hashable 프로토콜에 관련해선 추후에 다시 작성하도록 한다.

참고로, String 도 hashable을 따르기 때문에 set으로 구성할 수 있다.

이 Set을 있다는 것만 알았지 사용해본적이 없었으니.. 이런 상황에서 사용해야 한다는 걸 바로 깨닫지 못하고 저런 뻘짓을 할뻔했다...


# Code it

간단한 코드를 통해서 성능을 확인 해보자.

capacity가 reserve된 것과 그렇지 않은 기본 배열 변수를 선언하여 process time을 비교 해보자

~~~ swift

extension MiniPoiViewModel: Hashable {
    
    var hashValue: Int {
        return self.poiId?.hashValue ?? 0
    }
}
~~~

앞서 말했듯이 String은 hashable을 따르고, poiId는 hashValue를 리턴하기 때문에 Set으로 구성할 수 있다.

# 결론
오늘은 Set에 관련해서 알아보았다.
중복된 값이 없는 데이터를 원하는 상황에서 처리하는데에 적합한 자료구조임을 파악하였고, 이를 사용하기 위해선 어떠한 조건이 필요하다 라는것까지 알게 되었다.
앞으로도 Swift내의 자료구조를 공부하는데에 있어, 구체적인 예시와 함께 공부 해보면 지식을 체화하는데에 시간을 단축시킬 수 있을 거라 생각된다.