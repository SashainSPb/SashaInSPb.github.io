---
layout: post
title: "📅 Kotlin 강의노트-2"
excerpt: "JetBrains Academy에서 제공하는 Kotlin 교육 프로그램 강의 노트입니다."
subtitle: "Project: Simple Chatty Bot"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2021-12-22
tags: [Kotlin, JetBrains]
---

# Project: Simple Chatty Bot

## kotlin은 다음과 같은 특징을 갖고 있음

- 코틀린은 멀티플랫폼 언어
  JVM: JAVA 및 라이브러리와 100% 호환
  Android:모바일 app 구축 가능
  JS: client-side web 구동을 가능케 함
  Native: 플랫폼에 종속되어 있지 않음

- pragmatic language 실용적인 언어
  multiple programming paradigms
  tool-friendly language

## gradle

- 소스코드를 실행가능한 app 생성물을 자동으로 만들어주는 '빌드도구', 특히 gradle은 유연함과 성능에 초점을 뒀음.
  소스코드 컴파일 -> 패키징하여 실행 가능한 형태로 가공

## Basic literals(변수나 상수에 할당되는 그 데이터 자체를 뜻함)

- Integer numbers
  comma 대신 underscore를 넣어서 정수 자리수 구분하며, 정수의 맨 끝과 앞에 넣을 시 에러발생
  ex) 1_000_000 vs 1000000
- Characters
  홑따옴표로 감싸서 표시하며, 하나의 character만 표시할 수 있음(**)
  숫자, 글자, 그밖의 기호를 나타낼 수 있으며 보통 홑따옴표로 감싸서 표시한다 ex) '$'
- Strings
  텍스트 info를 나타내며, character의 집합임. character와 다르게 쌍따옴표로 감싸서 표시한다.
  string은 문자, 숫자, 공백, 그외의 character를 포함할 수 있다.
  홀따옴표와 쌍따옴표의 구분이 명확하다(**)
  ex) '1'은 character, and "1" strings.

## Terminology

  - program: 예측 가능한 방법으로 차례대로 실행되는 명령의 순서 모음
  - statement: 실행되는 싱글 명령
  - expression: 한 개의 value를 생성하는 코드
  - block: curly braces로 감싸져 있는 지역 단위
  - keyword: 프로그래밍 언어 내에 이미 정의되어 있는 단어, 변수나 상수 선언 시 피할 것 ex) fun
  - identifier: 프로그래머가 어떤 것을 식별하기 위해 쓰여진 단어
  - type inference: 입력된 변수의 데이터 타입을 자동으로 판별하는 기능, 타입을 따로 정의해줄 필요 없다.
  - type coercion: 타입 강제 변환

## Entry point

  - fun main(){...}: 단어 'fun'을 이용하여 함수 선언을 했고, main은 함수명. 그리고 함수명은 kotlin program의
  엔트리포인트를 나타낸다.
  - entry point: 프로그램 실행 시 시작되는 부분을 뜻하며, 보통 'main' 이라고 지정된 이름으로 시작될 수 있음.

## Standard output

  - println: string 다음에 새로운 줄을 스크린에 표시 <div>와 비슷
  - print: string을 한 줄에 다 표시. <span>과 비슷
