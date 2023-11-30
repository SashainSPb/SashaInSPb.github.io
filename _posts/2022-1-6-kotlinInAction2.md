---
layout: post
title: "📅 Kotlin in Action - 2장"
excerpt: "Kotlin in Action 2장 요약 노트입니다."
subtitle: "Kotlin in Action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-1-6
tags: [Kotlin]
---

## 2.1 기본 요소: 함수와 변수

#### 2.1.1 Hello, World!

  ```kotlin
  fun main(args: Array<string>) {ㅇ
      println("Hello, world!")
  }
  ```

  - 함수를 선언할 때에 fun 키워드를 사용
  - 인자 뒤에 타입을 사용
  - 함수를 최상위 수준에 정의할 수 있으며, 자바와 달리 꼭 클래스 안에 함수를 넣어야 할 필요가 없음
  - 배열 처리를 위한 문법이 따로 존재하지 않음
  - 표준 자바 라이브러리 함수를 간결하게 사용할 수 있게 감싼 래퍼를 제공. 덕분에 System.out.println 대신에 println라고 간결하게 사용 가능 
  - 세미콜론 생략가능

#### 2.1.2 함수

  ```kotlin
  fun max(a: Int, b: Int): Int {
      return if (a > b) a else b
  } // expression body: 식본문
  println(max(1,2)) // 2
  
  fun max(a: Int, b: Int): Int = if(a > b) a else b // block body: 블록본문
  ```

  - 코틀린에서 if는 자바와 다르게 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여할 수 있는 expression임
  참고로 statement는 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소이며, 어떤 값도 만들어내지 않음
  
  - type inference(타입추론): 식이 본문인 함수의 경우 반환 타입을 적지 않아도 컴파일러가 함수 본문 식을 분석해 결과 타입을 
  함수 반환 타입으로 정해주는 것을 뜻함. 식본문 함수에서만 타입 생략 가능

#### 2.1.3 변수

  - var: 변경 가능한 참조를 저장하는 변수. 자바의 일반 변수와 같다. 변수의 값을 변경할 수 있지만 변수의 
  타입은 고정돼 바뀌지 않음
  - val: 변경 불가능한 참조를 저장하는 변수. 초기화 후 재할당은 불가하지만 객체의 내부 값은 변경 가능
  자바의 final 변수와 같음
 
  ```kotlin
  val languages = arraylistOf("Java") 
  languages.add("Kotlin") // true
  
  var anwer = 42
  answer = "no answer" // Error: type mismatch
  ```

#### 2.1.4 더 쉽게 문자열 형식 지정: 문자열 템플릿

  - 문자열 리터럴 안에서 변수를 사용하기 위해서는 '$'를 앞에 붙여주자
  - 문자열 템플릿 안에는 간단한 변수뿐만이 아니라 식도 들어갈 수 있으므로 가능하면 '${변수 or 식}' 형태로 이용  
  - 또한 '$'를 특수문자로 쓰고 싶다면 '\'를 이용하여 escape 시키면 가능

  ```kotlin
  fun main(args: Array<String>) {
      if (args.size > 0) {
          println("Hello, ${args[0]}")
      }
  }
  
  fun main(args: Array<String>) {
      println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
  } //템플릿 리터럴 안에서 쌍따옴표도 사용 가능
  ```

### 2.2 클래스와 프로퍼티

  - 간단한 자바빈 클래스인 Person을 정의했으며, name 이라는 프로퍼티만 들어있음

    ```java
    public class Person {
        private final String name;
        public class Person(String name) {
            this.name = name;
        }
        public String getName() {
            return name;
        }
    }
    ```

  - 코틀린으로 치환한 코드이며 코드량이 **확연히** 적어진 것을 확인할 수 있음. 이렇게 코드없이 데이터만 저장하는 클래스를 value object(값 객체)라 부름

    ```kotlin
    class Person(val name: String)
    ```

#### 2.2.1 프로퍼티

  - 자바에서는 필드와 접근자를 묶어 **프로퍼티**라고 부르며, 코틀린은 기본 기능으로 제공하며 필드와 접근자를 완전히 대신함

    ```kotlin
    class Person(
        val name: String, // 읽기 전용 프로퍼티, 코틀린은 필드와 게터 생성
        val isMarried: Boolean // 쓸 수 있는 프로퍼티, 필드와 세터 그리고 게터 생성 
    )
  
    val person = Person("Bob", true)
    println(person.name) // Bob
    println(person.isMarried) // true
    ```

    > 필드: 데이터를 저장하는 곳  
    > 게터: 필드를 읽는 것   
    > 세터: 필드를 변경하는 것 
    > accessor method (접근자 메소드): 클라이언트가 클래스를 통해서 데이터에 접근할 수 있게 제공하는 통로

#### 2.2.2 커스텀 접근자

  - get(): 커스텀 프로퍼티 접근자이며, 클라이언트가 해당 접근자를 이용하여 프로퍼티 값을 계산
    
    ```kotlin
    class Rectangle(val height: Int, val width: Int) {
        val isSquare: Boolean
        get() { //프로퍼티 게터 선언
            return height == width
        }
    }
  
    val rectangle = Rectangle(41, 43)
    println(rectangle.isSquare) // false
    ```

#### 2.2.3 코틀린 소스코드 구조: 디렉토리와 패키지

  - 코틀린은 node.js 처럼 모듈화로 관리할 수 있고 이를 **패키지**라고 함. 그리고 이 패키지 안에는 클래스, 함수, 프로퍼티
    등이 있으며, 자유롭게 꺼내 쓸 수 있음

  - 프로젝트에서 자바와 혼용할 경우, 자바와 같이 패키지 별로 디렉터리를 구성

    ```kotlin
    package geometry.shapes // 패키지 선언
    import java.util.Random // 표준 자바 라이브러리 클래스를 import
    class Rectangle(val height: Int, val width: Int) {
        val isSquare: Boolean
            get() = height = width
    }
  
    fun createRandomRectangle(): Rectangle {
        val random = Random()
        return Retangle(random.nextInt(), random.nextInt())
    }
    ```

    ```kotlin
    pakage.* // 패키지 안의 모든 클래스, 함수, 프로퍼티까지 다 불러옴
    package geometry.example
    import geometry.shapes.createRandomRectangle // 이름으로 함수 import
    fun main(args: Array<String>) {
        println(createRandomRectangle().isSquare)
    }
    ```
  
### 2.3 선택 표현과 처리: enum과 when

#### 2.3.1 선택 표현과 처리 - enum 클래스 정의

  - enum에 생성자와 프로퍼티를 선언하고 클래스 안에 메소드를 정의하는 경우 반드시 상수 목록과 메소드 정의 사이에 세미콜론
  > enum - 복수의 요소를 나열한다는 점에서 JS의 배열과 유사하지만 프로퍼티나 메소드까지 정의 가능

    ```kotlin
    enum class Color (
        val r: Int, val g: Int, val b: Int // 상수 프로퍼티 정의
    ) {
        RED(255,0,0), ORANGE(255, 165, 0), // 상수 생성 시 이에 대한 프로퍼티 값 지정
        YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 255, 255); // 반드시 !세미콜론으로 마물
        
        fun rgb() = (r * 256 + g) * 256 + b // enum 클래스 안에서 메소드를 정의
    }
    ```

#### 2.3.2 / 2.3.3 when으로 enum 클래스 다루기

  - 분기를 생성하여 해당 조건에 맞는 값을 반환해주는 강력한 기능을 보유. 자바의 switch와 동일하나, 분기 조건에 상수만
    사용할 수 있는 swith와 달리 when의 분기 조건은 임의의 객체를 허용

    ```kotlin
    // when 분기 조건에 여러 값 사용하기
    fun getWarmth (color: Color) = when (color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
        Color.GREEN -> "neutral"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
    }
    println(getWarmth(Color.ORANGE)) // warm
  
    // when 분기 조건에 여러 다른 객체 사용하기
    fun mix(c1: Color, c2: Color) = 
        when (setof(c1,c2)) {
            setOf(RED, YELLOW) -> ORANGE 
            setOf(YELLOW, BLUE) -> GREEN
            setOf(BLUE, VOILET) -> INDIGO
            else -> throw Exception("FUCKED")
        }
    println(mix(BLUE, YELLOW)) // GREEN
    ```

  - setOf 함수는 인자로 전달받은 여러 객체를 포함하는 집합인 set 객체로 만들어 줌. 집합(set)은 원소가 모여있는 컬렉션으로 순서는 상관없음

#### 2.3.4 인자없는 when 사용

  - 앞 전의 함수는 호출될 떄마다 여러 Set 인스턴스를 생성하기 때문에 비효율성이 발생. 이에 다음과 같이 인자가 없는 when 식을
    사용하면 불필요한 객체(=가비지 객체) 생성을 막을 수 있음

    ```kotlin
    fun mixOptimized(c1: Color, c2: Color) = 
        when {
            (c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
            (c1 == YELLOW && c2 == BLUE) || (c1 == BLUE && c2 == YELLOW) -> GREEN
            (c1 == BLUE && c2 == VIOLET) || (c1 == VIOLET && c2 == BLUE) -> INDIGO
          
            else -> throw Exception("FUCKED")
        }
    println(mixOptimized()(BLUE, YELLOW)) // GREEN
  
    fun main() {
        println(doWhen(1))
    }
  
    fun doWhen (a: Any): String { // 출력값 타입을 지정하지 않은 경우 Type mismatch: inferred type is String but Unit was expected
        var result = when(a) {
            1 -> "1"
            "Sasha" -> "사샤"
            else -> "유효한 값을 입력하십시오"
        }
        return result
    }
    ```

#### 2.3.5 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

  - 스마트캐스트: 굳이 원하는 타입으로 캐스팅(형변환) 하지 않더라도, 컴파일러가 알아서 캐스팅 해주는 것

    ```kotlin
    fun eval(e: Expr): Int {
        if (e is Num) {
        val n = e as Num // 불필요한 타입 변환
        return n.value
        }
        if(e is Sum) { // 변수 e에 대해 형식 검사 및 자동으로 캐스팅 실행
            return eval(e.right) + eval(e.left)
        }
        throw IllegalArgumentException("본적없는 식인데?")
    }
    
    println(eval(Sum(Sum(Num(1), Num(2)), Num(4)))) // 7
    ```

#### 2.3.6 리팩토링: if를 when으로 변경

  - if 식을 이용한 값 생성
  - if나 when의 각 분기에서 수행해야 하는 로직이 복잡해지면 분기 본문에 블록 사용 가능

    ```kotlin
    fun eval(e: Expr): Int = 
        if (e is Num) {
            e.value
        } else if (e is Sum) {
            eval(e.right) + eval(e.left)
        } else {
            throw IllegalArgumentException("Unknown expression")
        }
    println(eval(Sum(Num(1), Num(2)))) // 3
    ```
    
  - if 대신에 when을 사용해서 좀 더 다듬어보자
    
    ```kotlin
    fun eval(e: Expr) : Int = 
        when (e) {
            is Num -> // 인자 타입을 검사한다 
                e.value
            is Sum -> 
                eval(e.right) + eval(e.left)
        else ->         
            throw IllegalArgumentException("Unknown expression")
        }
    ```
  
#### 2.3.7 if와 when의 분기에서 블록 사용

  - if나 when 모두 분기에 블록 사용이 가능하며, 그런 경우 **블록의 가장 마지막 문장이 블록의 전체 결과**가 됨. return을
  따로 써줄 필요가 없음.

    ```kotlin
    fun evalWithLogging(e: Expr) : Int = 
        when (e) {
            is Num -> {
                println("num: ${e.value}")
                e.value
            }
            is Sum -> {
                val left = evalWithLogging(e.left)
                val right = evalWithLogging(e.right)
                println("sum: ${left} + ${right}")
                left + right
            }
            else -> throw IllegalArgumentException("Unknown expression")
        }
    ```


### 2.4 대상을 iteration: while과 for loop

  - for는 자바의 for-each 루프에 해당하는 형태만 존재하며, 형태는 for <아이템> in <원소들>

#### 2.4.1 while 루프

  ```kotlin
  // 조건이 참인 동안 본문을 반복 실행
  while (조건) {
      /* 실행문*/
  }
  
  // 맨 처음에 무조건 본문을 한번 실행한 다음, 조건이 참인 동안 본문을 반복 실행
  do {
      /* 실행문*/    
  } while (조건)
  ```

#### 2.4.2 수에 대한 이터레이션: 범위와 수열

  - 코틀린에는 자바의 for loop에 해당하는 요소가 없으며, 이에 범위(range)를 따로 지정
  - 범위는 정수 등의 숫자 타입의 값이며 양끝을 포함

  ```kotlin
  val oneToTwenty = 1..20
  ```
  
  ```kotlin
  // when을 사용해 fizzBuzz 게임 구현하기
  fun fizzBuzz(i: Int) = when {
      i % 15 == 0 -> "FizzBuzz"
      i % 3 == 0 -> "Fizz"
      i % 5 == 0 -> "Buzz"
      else -> "$i"
  }
  // 증가 값을 갖고 범위 이터레이션
  for (i in 1..100) {
      print(fizzBuzz(i))
  }
  // downTo 1은 역방향 수열을 지정하며, step 2는 증가값(감소값)이 2(-2)를 뜻함 
  for (i in 200 downTo 1 step 2) {
      print(fizzBuzz(i))
  }
  // until은 끝 값을 포함하지 않는다
  for (i in 0 until size) {
      print(fizzBuzz(i))
  }
  ```

#### 2.4.3 맵에 대한 이터레이션

  - for loop를 사용해 이터레이션하려는 컬렉션의 원소를 풀 수 있음

  ```kotlin
  val binaryReps = TreeMap<Char, String>() 
  for (c in 'A'..'F') { // A부터 F까지 문자범위 사용
      val binary = Integer.toBinaryString(c.toInt()) // 아스키 코드를 2진 표현으로 변경
      binaryReps[c] = binary
  }
  for ((letter, binary) in binaryReps) {
      println("$letter = $binary")
      
  }
  
  // 객체를 표현하는 방법 2가지
  binaryReps [c] = binary
  binaryReps.put (c, binary)
  ```

#### 2.4.4 in으로 컬렉션이나 범위의 원소 검사

  - in 연산자를 사용해 어떤 값이 범위에 속하는지 알 수 있음. !in 연산자를 사용해 반대의 경우도 확인 가능

  ```kotlin
  fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
  fun isNotDigit(c: Char) = c !in '0'..'9'
  ```

  ```kotlin
  // when에서 in 사용하기
  fun recognize(c: Char) = when (c) {
      in '0'..'9' -> "It's a digit"
      in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
      else -> "Fucked"
  }
  ```
  - 비교가 가능한 클래스라면 그 클래스의 인스턴스 객체를 사용해 범위 생성 가능

### 2.5 코틀린의 예외처리

- 발생한 예외를 함수 호출단에서 **catch**하지 않으면 함수 호출 스택을 거슬러 올라가면서 예외를 처리하는 부분이 나올 때까지 예외를 다시 던짐

  ```kotlin
  val percentage =
      if (number in 0..100)
          number
      else
          throw IllegalArgumentException (
              "A percentage value must be between 0 and 100: $number")
  ```
  
#### 2.5.1 try, catch, finally

  - 자바와 마찬가지로 파일에서 각 줄을 읽어 수로 변환하되 그 줄이 올바른 수 형태가 아니면 null 반환
  - 체크 예외 처리를 강제하는 자바와는 달리 코틀린은 자유로움
  
  ```kotlin
  fun readNumber (reader: BufferReader) : Int? {
      try {
          val line = reader.readLine()
          return Integer.parseInt(line)
      }
      catch (e: NumberFormatException) {
          return null
      }
      finally {
          reader.close()
      }
  }
  ```

#### 2.5.2 try를 식으로 사용

  - 코틀린의 try 키워드는 식이므로 변수에 대입할 수 있고, 본문을 반드시 중괄호 {}로 둘러싸야함
  - JS의 try catch 구문과 거의 유사

  ```kotlin
  // try를 식으로 사용하기
  fun readNumber(reader: BufferedReader) {
      val number = try {
          Integer.parseInt(reader.readLine()) // try 식의 값
      } catch (e: NumberFormatException) {
          return 
      }
      println(number)
  }
  
  // catch에서 값 반환하기
  fun readNumber(reader: BufferedReader) {
      val number = try {
          Integer.parseInt(reader.readLine())
      } catch (e: NumberFormatException) {
          null // 예외가 발생하면 null값을 사용 
      }
  }
  ```
