---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍 - 기본편"
excerpt: "자바 ORM 표준 JPA 프로그래밍 강의노트 요약입니다."
subtitle: "JPA"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-6-30
tags: [JPA]
---


#### 수신객체

- 클래스 내 변수처럼 프로퍼티를 사용하기 위해서, 사용되는 인스턴스를 식으로 지정하는 객체를 칭함
- 하기 코드에서 person을 수신객체라 칭하며, this로 해당 객체를 참조할 수 있음
- 자바와 달리 코틀린의 클래스 가시성은 public 
- 

    ```kotlin
    fun main() {
        var person = Person() // 생성자 호출을 통해 인스턴스 생성
        person.showMe()
        person.fullName(person)
        
    }
    
    class Person {
        var firstName: String = "Sasha"
        var familyName: String = "Park"
        var age: Int = 0
        
        fun fullName(person: Person) = println(person.firstName) // person 수신 객체
        fun showMe() {
            println("${this.firstName} ${this.familyName}") // this로 참조 가능
        }
    }
    
    ```
  
#### 생성자

- 클래스 인스턴스를 초기화해주고 인스턴스 생성 시 호출되는 함수

    ```kotlin
    fun main() {
        var person = Person("Sasha Jung")
        println(person.firstName)
        println(person.familyName)               
    }
    
    // class header의 파라미터 목록을 primary constructor 선언이라고 칭함
    class Person(fullName: String) {  
        val firstName: String 
        val familyName: String
            
        // 초기화 블록이며, return 문이 들어가지 못함
        init { 
            val names = fullName.split(" ")
            if(names.size != 2) throw IllegalArgumentException("invalid name: ${fullName}")
            firstName = names[0] // 복잡한 초기화 로직을 실행시키기 위해서 init block 내에서 프로퍼티 초기화가 가능
            familyName = names[1]
            println("Created new Person instance: ${fullName}")
        }
    }
    ```
  
- 주생성자의 모든 실행 경로가 모든 멤버 프로퍼티를 초기화하거나 예외를 발생시키는지 확인이 안된다면 컴파일러는 다음과 같은 오류 발생 시킴
- error: property mut be initialized or be abstract
- 주생성자 파라미터를 프로퍼티 초기화나 init 블록 밖에서 사용할 수 없음.

    ```kotlin
    fun main() {
        var person = Person("Sasha", "Park")
        person.printFirstName() 
        
    }
    
    // 초기화 방법 1. 멤버 프로퍼티 정의
    class Person1(val firstName: String, familyName: String) { 
        val fullName = "${firstName} ${familyName}"
        val firstName = firstName // 2. 주생성자 파라미터를 저장할 멤버 프로퍼티를 정의
        fun printFirstName() {
            println(firstName) // 1. firstName 참조를 찾을 수 없는 오류 발생
        }    
    }  
    
    // 초기화 방법 2. 주생성자 파라미터 내 변수 선언
    class Person2(val firstName: String, val familyName: String) {
    fun fullName() = "${firstName} ${familyName}"
    } 
    
    // 초기화 방법 3. constructor(부 생성자)를 통한 초기화 
    class Person3 {
	    val firstName: String 
        val familyName: String
    
    constructor(firstName: String, familyName: String) {
        this.firstName = firstName
        this.familyName = familyName
    }
    
    // 초기화 방법 3-1. 생성자 위임 호출을 사용하여 다른 부생성자를 호출
    constructor(fullName: String): this("${fullName}")
  
    }
    ```
  
    