---
layout: post
title: "Array reserveCapacity"
description: "Swift의 reserveCapacity를 통해 성능향상을 노려보자"
categories: Programming
tags: [programming, recursive, swift]
redirect_from:
   - /2018/08/26
---

# At a Glance

array를 mutable하게 사용하고 요소를 추가할 때, array엔 필요한 capacity가 추가가 된다.
Swift에선 배열의 요소를 추가하기 위한 배열 용량 할당에 exponential growth startegy을 사용하여 배열의 메모리를 할당한다.

# Exponential growth strategy
Swift에서 사용하는 배열의 추가 할당 전략으로써, 기존 예약된 사이즈를 넘겼을 때, 이를 지수곱으로 할당하여 요소 추가시의 작업을 평균화 하여 요소 추가 할당 시간을 일정 시간에 발생할 수 있게 만들어 놓았다.
이런 추가 작성 작업에도 시간이 소요가 되기 때문에 이를 단축시켜 성능 향상을 기대해 볼 수 있다.

# Reserve Capacity
할당 시간을 단축시키는 방법은 할당 횟수를 축소 시키는 것이 방법이 된다.
배열에 할당이 될 minimum capacity를 먼저 예약 하여 추가 배열을 저장할 때, 이미 할당된 배열에 추가로 할당 되는 시간을 축소 시켜 성능 향상을 도모하는 것이다.

Swift array에선 reserveCapacity 메소드가 있어, minimum capacity를 통해 메모리 할당 선작업을 하고 배열에 새로운 요소를 추가할 수 있게 해준다.

# Code it

간단한 코드를 통해서 성능을 확인 해보자.

capacity가 reserve된 것과 그렇지 않은 기본 배열 변수를 선언하여 process time을 비교 해보자

~~~ swift
var reserveIntArray = Array<Int>()
reserveIntArray.reserveCapacity(500)
reserveIntArray.capacity

var nonReserveIntArray = [Int]()
nonReserveIntArray.capacity

processTime {
    for i in 0...500 {
        reserveIntArray.append(i)
    }
    reserveIntArray.capacity
}
processTime {
    for j in 0...500 {
        nonReserveIntArray.append(j)
    }
    nonReserveIntArray.capacity
}
~~~

결과 값은 아래와 같다.
~~~
processTime: 0.620948076248169
processTime: 0.7575379610061646
~~~


기존 할당한 capacity내에서의 배열의 추가 성능은 0.13sec 차이가 나왔다. 생각보다 많은 차이가 나서 놀라긴 했는데.. 몇번 더 테스트를 해보면 그 차이는 일정하게 나타나진 않는다.
하지만 당연히 reserve한 배열이 그렇지 않은 배열보다 배열의 추가성능이 빠른 것은 자명했다.

# One More
 한 가지가 더 궁금했다. 그럼 예약 된 capacity에서 초과된 작업을 진행할 때엔 어떤 결과가 나올까

~~~ swift
var reserveIntArray = Array<Int>()
reserveIntArray.reserveCapacity(500)
reserveIntArray.capacity

var nonReserveIntArray = [Int]()
nonReserveIntArray.capacity

processTime {
    for i in 0…1000 {
        reserveIntArray.append(i)
    }
    reserveIntArray.capacity
}
processTime {
    for j in 0…1000{
        nonReserveIntArray.append(j)
    }
    nonReserveIntArray.capacity
}
~~~
결과 값은 아래와 같다.

~~~
processTime: 1.640960931777954
processTime: 1.3353140354156494
~~~


예약된 배열의 capacity에서 2배 수치의 요소를 삽입하였을 때 기존 예약되지 않은 배열보다 월등히 느린것이 확인 되었다.

혹시나 하여 capacity를 확인해보니, 2배의 수치가 일어났을 때 지수 증가 법칙으로 인해 500의 2배인 1000이 할당되었고,
용량 예약이 되지 않은 array는 1500이상이 할당 되었다.
그 이전 capacity가 508, 768 이였기 때문에 할당 횟수에 대한 차이는 크게 없는 듯 하다.
원인에 대해선 좀 더 확인해봐야겠지만 capacity를 통해 성능 향상을 기대하기 위해선 예약 수치에 대한 규정을 명확하게 하여 사용할 것을 권장한다.

# 결론
기존에 페이징이 필요한 tableView에서 해당 배열을 추가하여 데이터를 표시하는 작업을 진행했다.
자연스레 추가 할당에 대한 시간을 단축 시킬 수 있다면 이에 대한 성능 향상을 기대해 볼 수 있다.
리스트에 나타나는 최대 갯수, 페이징의 단위를 명확하게 규정된 UI를 구성할 때 유용하게 사용해 볼 수 있을 것으로 기대된다.
