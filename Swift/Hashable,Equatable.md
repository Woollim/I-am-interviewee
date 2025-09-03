# Hashable이 무엇이고, Equatable을 왜 상속해야 하는지 설명하시오.

### 참고

[Apple Developer - Hashable](https://developer.apple.com/documentation/swift/hashable/)

## Anser

### Q1. Hashable이란?

공식문서 내용을 그대로 가지고 오면, 정수형 hashValue를 생성하기 위해 Hasher로 해싱될 수 있는 유형을 말한다.

이게 무슨 말일까? 윗 글을 온전히 이해하기 위해 아래 추가 질문에 하나씩 답을 해보자.


#### Q1-1. hashing이란?

입력값을 특정한 수학적 알고리즘을 통해 고정된 크기의 값(hashValue)으로 변환하는 동작을 말한다.

해시 함수의 주요한 특징은 단방향 함수라는 것인데, 입력값을 hashValue로 변환할 수는 있지만 반대로 hashValue를 다시 입력값으로 원상복귀시킬 수 없다.

이러한 특징 덕분에 보안에서 값을 유추할 수 없도록 만들어야 할 때 hashing을 사용한다. ex) 비밀번호를 저장해야 할 때 hasing을 사용한다. 이러면 DB가 털린다고 해도 해커는 비밀번호를 알아낼 수 없다.

hasing이라는 개념이 입력값을 수학적 알고리즘을 통해 hashValue로 바꾸는 행위를 말하며 이를 구현한 hasing 알고리즘은 다양하다. (hashMap, SHA256 등)

#### Q1-2. swift가 hasing을 하는 방법은? (Hasher는 어떤 방식으로 hashValue를 만드는가?)

그렇다면 과연 swift는 어떤 hasing 알고리즘을 사용할까?

[swift 공식 레포](https://github.com/swiftlang/swift/blob/1c8f885a6d57cb9027e0a52070caf68586abc857/stdlib/public/core/SipHash.swift)와 [포럼](https://forums.swift.org/t/how-is-synthesized-hashable-implemented/15960?utm_source=chatgpt.com)에 명시되어 있는 것 처럼 `SipHash`를 사용하고 있다.

이 알고리즘을 사용하는 swift Hasher의 특징은 아래와 같다.

1. 같은 입력값에 같은 hashValue가 나오는 consistent hashing 방식이라는 점
2. 내부적으로 hasing을 위해 사용하는 seed값 2개를 프로세스마다 무작위로 잡는다는 것
3. 동일한 값을 hasing하더라도 값을 넣는 순서에 따라 hashValue가 다를 수 있다는 점
4. 서로 다른 입력값을 hasing 하더라도 같은 hashValue가 나올 수 있다는 점

#### Q1-3. `hash(into: Hasher)` 를 직접 구현할 때 유의사항은?

1. 프로세스에 따라 서로 다른 seed값을 잡기 때문에 hashValue가 계속 변할 수 있다는 점
   - 다른 앱의 hashValue와 비교 불가능
   - 앱을 껐다켜도 hashValue가 변하기에 저장도 불가능
2. 서로 다른 타입이더라도 hashing을 하는 입력값이 동일하면 같은 hashValue가 나올 수 있다는 점
3. 입력값이 동일하더라도 hashing을 하는 순서가 다르다면 다른 hashValue가 나올 수 있다는 점



### Q2. 그렇다면, Hashable은 왜 Equatable을 상속해야할까?

swift는 Hashable을 언제 사용할까? 값을 해시(hashValue) 기반으로 빠르게 찾거나 중복을 막아야 할 때 사용한다.

Set, Dictionary, DiffableDataSource 등 Hashable을 사용하면 요소들을 순서대로 돌면서 `equal` 체크하는 것보다 해시에 지정된 값을 바로 꺼낼 수 있기 때문이다. 즉 시간복잡도를 O(n)을 O(1)로 만든다.

하지만 해시(hashValue) 기반으로 빠르게 찾아야 하는데, Hasher의 특징에 하나 걸리는 것이 있다.

> 그리고 이 Hasher의 특징으로 서로 다른 입력값을 hasing했다고 하더라도 같은 hashValue가 나올 수 있다는 점이 있다.


즉 hashValue만으로는 값의 동등성을 완전히 식별하지 못한다는 것이다. 이를 [해시충돌](https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%EC%B6%A9%EB%8F%8C) 이라고 부르며, swift는 해시충돌 상황에서 값의 동등성을 완전히 식별하기 위해 equal 함수를 추가로 사용하고 있다.


**한마디로 정리하면**

swift의 hash함수(SipHash)으로 인해 해시충돌이 발생할 수 있고, 이 상황에서도 값의 동등성을 식별하기 위해 Equatable을 필수로 구현해야한다.



PS. [DiffableDataSource의 문서](https://developer.apple.com/documentation/uikit/updating-collection-views-using-diffable-data-sources)에도 같은 내용을 서술하고 있다.

> Two identifiers that are equal must always have the same hash value. However, the converse isn’t true; two values with the same hash value aren’t required to be equal. This situation is called a *hash collision*. To increase efficiency, try to ensure that unequal identifiers have different hash values. The occasional hash collision is okay when it’s unavoidable, but keep the number of collisions to a minimum. Otherwise, the performance of lookups in the data collection may suffer.



