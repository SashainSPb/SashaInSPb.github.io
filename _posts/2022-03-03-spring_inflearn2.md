---
layout: post
title: "📅 Spring 강의노트"
excerpt: " Spring 핵심 원리에 대한 강의 ②"
subtitle: "스프링 핵심 원리 - 기본편"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-3-3
tags: [Spring]
---

#### 스프링으로 전환하기

  - 'ApplicationContext'를 스프링 컨테이너라고 함
  - '@Bean'이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록하며, 등록된 객체를 스프링 빈이라고 칭함
  - 스프링 bean은 메서드명을 자체 이름으로 정하며, applicationContext.getBean() 메서드를 사용해서 찾고 이용 가능  

    > 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너(저장소)에서 빈을 찾아서 사용.
    > 빈 이름 등록 시 설정 오류를 피하기 위해 항상 다른 이름을 부여 필요
  

#### 스프링 빈 조회

  - 이름으로 빈 조회: ac.getBean(name: "BeanName", BeanName.class)
  - 이름 없이 타입으로만 조회: ac.getBean(BeanName.class)
  - 구체 타입으로 조회: ac.getBean(name: "BeanName", BeanNameImpl.class) -> 변경 시 유연성이 떨어짐
  - 동일한 타입이 둘 이상: ac.getBeansOfType(BeanName.class)

  - 상속관계: 부모 타입 조회 시, 자식 타입도 함께 조회. 최상위 객체인 'Object' 타입으로 조회 시, 모든 스프링 빈을 조회

#### BeanFactory & ApplicationContext: 스프링 컨테이너 

  - BeanFactory: 스프링 컨테이너의 최상위 인터페이스
    - 스프링 빈을 관리하고 조회하는 역할 담당 
    - getBean()을 제공하며, 대부분의 기능을 제공  
  
  - ApplicationContext: BeanFactory 기능을 모두 상속 받아서 제공하며, 다음과 같은 부가 기능을 제공
    - MessageSource: 메세지 소스를 활용한 국제화 기능
    - EnvironmentalCapable: 로컬, 개발, 운영 등을 구분해서 처리
    - ApplicationEventPublisher: 이벤트를 발행하고 구독하는 모델을 편리하게 지원
    - ResourceLoader: file, classPath, 외부 등에서 리소스를 편리하게 조회

#### 다양한 설정 형식 지원 - 자바코드, XML

  - 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연하게 설계
  - 현재는 @을 이용하여 자바 코드로 설정을 생성하는 방법을 이용
  - XML은 HTML tag와 사용방법이 유사

#### 스프링 빈 설정 메타 정보 

  - BeanDefinition 이라는 '추상화' 혹은 인터페이스를 통해 다양한 설정 형식 지원하며 다음과 같은 프로퍼티를 지님
    - BeanClassName: 생성할 빈의 클래스명 (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)
    - factoryBeanName: 팩토리 역할의 빈을 사용할 경우
    - factoryMethodName: 빈을 생성할 팩토리 메서드 지정 
    - Scope: 싱글톤(default)
    - lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때까지 최대한 생성을 지연처리 여부
    - InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
    - DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
    - Constructor arguments. Properties: 의존관계 주입에서 사용 (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)

#### 웹 어플리케이션과 싱글톤 패턴

  - 요청이 올때마다 객체를 생성하고 해당 객체가 메모리에 남아있다면 심각한 리소스 낭비 -> 해당 객체가 1개만 생성되고, 공유한다면 해결 가능
  - 싱글톤 패턴: 클래스의 인스턴스가 오직 **1개**만 생성되는 것을 보장하는 디자인 패턴 -> scope를 private으로 설정

  - 스프링 컨테이너는 기본적으로 싱글톤 패턴으로 객체 생성/관리 가능
  - 단, 다음과 같은 문제가 있음
    - 싱글톤 패턴 구현 코드가 많음
    - DIP 위반
    - 유연성이 떨어짐
    - 안티패턴
  
  - 싱글톤 컨테이너: 기존 스프링 컨테이너가 해당 역할을 가지고 있으며, 생성하고 관리하는 기능은 싱글톤 레지스트
  - 하나의 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지하게 설계하면 안됨 -> stateless
  - **스프링 빈의 필드에 공유 값을 설정하면 큰 장애 발생!**

#### @configuration과 싱글톤

- CGLIB:  