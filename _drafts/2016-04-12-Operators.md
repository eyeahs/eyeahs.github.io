---
published: false
---

http://reactivex.io/documentation/operators.html

// Operate - 운용, operation - 작업, apply - 적용, work - 동작
// in a chain 연쇄적, chain - 연쇄 

# Introduction
ReactiveX의 각 언어-특정 구현(Language-specific implementation)은 operators 세트를 구현한다. 구현들간에 겹치는 것이 많이 있지만 특정 구현들에서만 구현되는 operator들 역시 존재한다. 그리고 각 구현은 그 언어의 유사한 메소드들과 문맥상 이미 친숙한 닮은 이름을 짓는 경향이 있다.

## Chaining Operators
대부분의 operator는 Observable에서 운용되고 Observable을 반환한다. 이는 operator들을 하나 그 뒤에 또 하나, 연쇄(chain)적으로 적용할 수 있게 한다. 연쇄(chain) 내부의 각 operator는 이전 operator의 작업 결과인 Observable을 변경한다. 빌더 패턴처럼 특정 클래스의 다양한 메소드들이 동일 클래스의 한 항목의 객체를 메소드의 작업을 통해 수정함으로서 그 항목을 처리하는 다른 패턴들도 존재한다.(There are other patterns, like the Builder Pattern, in which a variety of methods of a particular class operate on an item of that same class by modifying that object through the operation of the method.) 이런 패턴들은 메소드를 비슷한 방식으로 묶을(chain) 수 있게 해준다. 하지만 빌더 패턴에서는 보통 연쇄 내에서 메소드가 나타나는 순서는 중요하지 않지만 Observable operator의 순서는 중요하다.

## The Operators of ReactiveX
이 페이지는 먼저 ReactiveX의 "핵심" operator로 고려할 수 있는 것들을 열거하고 이 operator들이 어떻게 동작하는지와 개별 언어-특정 ReactiveX 버전에서 어떻게 구현 되었는지에 대한 더 상세한 정보를 가진 페이지로 연결한다. 
다음은 "의사 결정 분지도(decision tree)"이다. 이는 당신의 유스케이스(use case)에 가장 적절한 operator를 선택하는 것을 도울 수 있을 것이다.
마지막은 많은 ReactiveX의 언어-특정 구현에서 사용할 수 있는 대부분의 operator들의 알파벳순 목록이다. 이것들은 언어-특정 operator와 가장 근접하게 닮은 핵심 operator을 기록하는 페이지에 연결한다. (예를 들어, Rx.NET "SelectMany" operator는 FlatMap ReactiveX operator의 문서에 연결되고, 이 문서의 "SelectMany"는 Rx.Net 구현이다.)

만약 당신만의 operator를 구현하기를 원한다면 [Implementing Your Own Operator](http://reactivex.io/documentation/implement-operator.html)를 보라.


### Contents
1. Operator 카테고리 (Operators By Category)
2. Observable Operator의 의사 결정 분지도 (A Decision Tree of Observable Operators)
3. Observable Operator의 알파벳순 목록 (An Alphabetical List of Observable Operators)

# Operator 카테고리 (Operators By Category)
## Observable 만들기 (Creating Observables)
새로운 Observable을 만드는(originate) Operator들
- [Create](http://reactivex.io/documentation/operators/create.html) - Observer 메소드들을 프로그램 코드로(programmatically) 호출함으로서 Observable을 새로 (from scratch) 생성한다.
- [Defer](http://reactivex.io/documentation/operators/defer.html) - Observer가 subscribe하기 전에는 Observable을 만들지 않으며, 각 observer마다 새로운(fresh) Observable을 생성한다.
- [Empty/Never/Throw](http://reactivex.io/documentation/operators/empty-never-throw.html) - 


