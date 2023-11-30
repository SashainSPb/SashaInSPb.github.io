---
layout: post
title: "📅 Kotlin in Action - 1장"
excerpt: "Kotlin in Action 1장 요약 노트입니다."
subtitle: "Kotlin in Action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-3-14
tags: [Kotlin]
---


### 1.1 코틀린 맛보기

  - kotlin: 간결하고 실용적이며, 자바 코드와의 상호운용성을 중시하는 새로운 프로그래밍 언어 
  - 코틀린의 특성을 잘 나타내는 하기 코드를 살펴보자

    ```kotlin
    // 데이터 클래스, null이 될 수 있는 타입과 전달인자 디폴트 값은 null로 설정
    data class Person(val name: String, val age: Int? = null)
    
    // 최상위 함수
    fun main(args: Array<String>) {
        val persons = listOf(Person("사샤"), Person("리사", age = 3))
        val oldest = persons.maxBy { it.age ?: 0 } // 람다식과 엘비스 연산자
        println("나이가 가장 많은 사람: $oldest") 
    }
    ```
    

#### 1.2.2 정적 타입 지정 언어


- statically typed(정적 타입): 모든 프로그램 구성 요소의 타입을 컴파일 시점에 알 수 있고 프로그램 안에서 객체의 필드나 메서드를 
사용할 때마다 컴파일러가 타입을 검증. 코틀린은 정적 타입 지정 언어이며 타입 추론을 통해 변수 타입을 자동으로 유추
- dynamically typed(동적 타입): 매서드나 필드 접근에 대한 검증이 실행 시점에 발생
- nullable type 지원: 코틀린은 컴파일 시점에 null pointer exception 이 발생할 수 있는지 여부를 검사할 수 있어 프로그램 신뢰성을 높일 수 있음


#### 1.2.3 함수형 프로그래밍과 객체지향 프로그래밍
 

- 함수형 프로그래밍 
  - first-class function: 함수를 변수에 저장하거나 다른 함수에 인자로 전달 혹은 함수에서 새로운 함수를 만들어서 반환하는 등 함수를 일반
  값처럼 다룰 수 있음
  - 불변성: 일단 만들어지고 나면 내부 상태가 절대로 바뀌지 않는 불변 객체를 사용해 프로그램 작성
  - 부수 효과 없음: 입력이 같으면 항상 같은 값을 반환하고 다른 객체의 상태를 변경하지 않으며, 함수 외부나 다른 바깥 환경과 상호작용하지
  않는 순수 함수를 사용

  - 함수형 프로그래밍의 장점
    - 간결성, 더 강력한 추상화를 통해 코드 중복을 피함
      ```kotlin
      fun findSasha() = findPerson{ it.name == "Sasha" }
      fun findLisa() = findPerson{ it.name == "Lisa" }
      ```
    - 다중 스레드 사용시 안전함: 불변 데이터 구조를 사용하고 순수 함수를 데이터 구조에 적용한다면 같은 데이터를 여러 스레드가 변경X
    - Test 용이성: 전체 환경을 구성하는 준비 코드가 필요없음

  - 코틀린의 철학
    - 실용성: 연구를 위한 것이 아닌 프로젝트 내 문제를 해결하기 위해 만들어짐
    - 간결성: 생산성을 향상시켜주고 개발을 더 빠르게 진행할 수 있게 도와줌
    - 안전성: 컴파일 시점 검사를 통해 오류를 방지 ex) NullPointerException, ClassCastException
    - 상호운용성: 기존 자바와 혼합적으로 운용이 가능할 만큼 호환성을 높임 

#### 1.5.1 코틀린 코드 컴파일

- 코틀린 빌드과정 이미지 파일 업로드 요망
