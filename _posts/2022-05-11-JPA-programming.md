---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍 - 기본편"
excerpt: "자바 ORM 표준 JPA 프로그래밍 강의노트 요약입니다."
subtitle: "JPA"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-5-11
tags: [JPA]
---

#### Hello JPA - 프로젝트 생성

  - JPA는 특정 DB에 종속적이지 않게 설계되었음
  - 각각의 DB가 제공하는 SQL 문법과 함수는 조금씩 다름
    - MySQL은 LIMIT, Oracle은 ROWNUM 
  - 방언:SQL 표준을 지키지 않는 특정 DB만의 고유한 기능 
  - 엔티티의 생명주기
  - 영속상태: em.persist(member) DB에 저장되지는 않으며 트랜잭션 커밋 시(tx.commit())에 저장
  - 준영속: em.detach(member) 
  - 삭제: em.remove(member)


### EntityManager (영속성 컨텍스트)

#### EntityManager(영속성 컨텍스트)의 이점

  - 1차 캐시
  - identity 동일성 보장
  - 트랜잭션을 지원하는 쓰기 지연
  - 변경 감지 dirty checking
  - 지연 로딩 lazy loading 것

#### 엔티티 조회 

  - 1차 캐시(영속성 컨텍스트라고 생각할 것)에 저장, 한번 조회한 거 또 조회하면 DB에 쿼리 안날림.

    ```kotlin
      try {
        // 비영속
        Member member = new Member();
        member.setId(100L);
        member.setName("HelloJPA");
        // 영속
        em.persist(member)
        // 엔티티 삭제
        em.remove(member)
      } catch (Exception e) {
        tx.rollback();
      } finally {
        tx.commit()    
      }
    ```
  - 영속 엔티티의 동일성 보장: 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 DB가 아닌 application 차원에서 제공
  - 쓰기 지연 SQL 저장소
  - jdbc.batch: 버퍼링 기능, commit을 모아서 한번에 push 하는 기능? 
  - Dirty checking (변경 감지): 굳이 DB에 update 쿼리를 안날리고 값만 변경해도 알아서 저장해주는 기능
    - flush(): 영속성 컨텍스트의 변경 내용을 DB에 반영
    - 엔티티와 스냅샷 비교
    - Update sql 생성
    - flush
    - commit

#### Flush
  
  - Flush: 영속성 컨텍스트의 변경 내용을 DB에 반영
  - FlushModeType.Auto가 기본값이며, 이는 커밋이나 쿼리를 실행할 때 플러쉬 진행
  - 영속성 컨텍스를 비우지 않음
  - 영속성 컨텍스트를 플러시하는 방법
    - em.flush()
    - em.commit()
    - JPQL 쿼리실행 
  
  > **트랜잭션**이라는 작업 단위가 중요하며, 커밋 직전에만 동기화하면 됨
    
#### 준영속 상태

  - 영속 상태의 엔티티가 영속성 컨텍스트에서 detached(분리)된 상태

    ```kotlin
    em.detach(memeber) // 특정 엔티티만 준영속 상태로 전환
    em.clear() // 영속성 컨텍스트를 완전히 초기화
    em.close() // 영속성 컨텍스트를 종료
    ```

### 객체와 테이블 맵핑

#### 엔티티 맵핑

  - 객체와 테이블 맵핑: @Entity, @Table
    - @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 칭함
    - 기본 생성자 필수 
    - final class, enum, interface, inner class 사용 X
    - 저장할 필드에 final 사용 X
  
  - 필드와 컬럼 맵핑: @Column
  - 기본 키 맵핑: @Id
  - 연관관계 맵핑: @ManyToOne, @JoinColumn

#### DB Schema 자동 생성

  - DDL: 데이터베이스의 테이블의 생성, 변경, 삭제를 담당하는 명령어
    - DDL을 어플리케이션 실행 시점에 자동 생성
    - 테이블에서 객체 중심으로 변경
    - DB 방언을 활용해서 DB에 맞는 적절한 DDL을 생성 후, 개발 환경에서만 사용 
    - **생성된 DDL은 가능하면 운영 서버에서 사용하지 말자** 

  > ddl-auto option: 운영 장비에는 절대 create, update(DB lock), create-drop 사용하면 안됌.

#### 필드와 컬럼 맵핑

  - @Transient: 메모리에서만 임시로 사용 
  - @Column: nullable, insertable, updatable
  - @Lob: 대용량의 데이터 타입 BLOB, CLOB
  - @Unique: @Table(uniqueContraints)로 보통 사용
  - @Enumerated: String은 이름을 DB에 저장, Ordinal은 enum의 순서를 DB에 저장하며 사용 시, 기존 데이터와 호환이 되지 않아 운영상의 위험 존재. String을 쓰자
  - Temporal: **deprecated**, LocalDate, LocalDateTime 사용

#### 기본 키 맵핑

  - @Id: 아이디를 직접 할당할 경우 사용 
    - @GeneratedValue: IDENTITY(=autoincrement)는 기본키 생성을 DB에 위임, sequence는 차례대로 생성,
    - Table 전략: 키 생성 전용 table을 만들어 DB 시퀀스를 흉내내는 전략. 모든 DB에 적용 가능하나 성능이 떨어지는 단점 보유 

    - 기본키: null 아님, 변하면 안된다. 하지만 먼 미래까지 해당 조건을 만족하는 자연키는 찾기 어려움으로 대리키(대체키) 사용을 권장 
    - 권장하는 식별자 전략: Long형 + 대체키 + 키 생성전략 사용 필요
    - AUTO_INCREMENT는 DB에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
    - SEQUENCE 전략 
      - Allocation size 옵션 -> DB에 미리 올려놓고 메모리에서 필요한 만큼 가져다 쓰는 전략

#### 연관관계 맵핑 기초

  - 객체의 참조와 테이블의 **외래키를 맵핑하는 것이 포인트
  - terminology
    - direction(방향): 단방향, 양방향
    - Multiplicity(다중성): 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)의 이해
    - Owner(연관관계의 주인): 객체 양방향 연관관계는 주체가 필요

> ***객체를 테이블에 맞추어 '데이터 중심'으로 모델링하면, 협력 관계를 만들 수 없음***

#### 단방향 연관관계

    ```kotlin
    @Entity
    class Team {
        @Id @GeneratedValue
        var id: Long
        @Column
        var name: string
        @Column
        var teamId: Long
        @ManyToOne(fetch = FetchType.LAZY)
        var team: Team? = null
    }
    ```

#### 양방향 연관관계와 연관관계의 주인 (매우 중요 밑줄 쫙!)

  - 테이블의 연관관계에서는 FK로 양방향 연관관계가 설정되어 있으나 객체는 그렇지 않음
  - 객체와 테이블이 관계를 맺는 차이
    - 객체의 연관 관계는 **서로 다른 2개의 단방향**으로 구성, 테이블의 연관관계는 1개
    - 그러면 하나의 데이터가 바뀌면 둘 데이터 중 하나를 바꿔야하는데 어떤 걸 바꿔야 하나? -> 연관관계의 주인
    - Owner(연간 관계의 주인): 객체의 두 관계 중 하나를 주인으로 지정
      - 주인만이 외래 키를 관리(등록, 수정)
      - 주인이 아닌쪽은 읽기만 가능, mappedBy 속성이 있음
      - 주인은 **mappedBy** property 사용 X
      - 리스트가 아닌 **외래키**가 있는 엔티티를 주인으로 정할 것. Many 쪽을 주인으로 설정하면 됨
  
  > **JPA 모델링 시, '단방향'으로 설계를 끝낼 것. 양방향은 필요할 때 추가(역방향 탐색, JPQL)**
  
  ```kotlin
    @Entity
    class Team {
        ...
        @OneToMany(mappedBy = "team")
        var userList: MutableSet<User> = LinkedHashSet()
    }
    
    class Member {
        @ManyToOne // 연관 관계의 주인
        var team: Team? = null
        ...
    }
  ```
  - 양방향 맵핑 시 가장 많이 하는 실수
    - 연관 관계의 주인에 값을 입력하지 않음 -> 양쪽 다 값을 입력하자
    - 연관관계 편의 메서드를 생성
    - 무한 루프(stack overflow)를 조심 
      - toString(), lombok, JSON 생성 라이브러리 

  ```kotlin
    @Entity
    class Team {
    ...
    @OneToMany(mappedBy = "team")
    var userList: MutableSet<User> = LinkedHashSet()
    }
            
    class Member {
      @ManyToOne // 연관 관계의 주인
      var team: Team? = null
      ...
    }
            
    class Orders {
      @JoinColumn(name = "MEMBER_ID")       
      var member: Member? = null
      ...    
    }
  ```