# [1/10] Android (안드로이드, 코틀린, 프로젝트 폴더 구조, 코틀린 기본 문법)

## 안드로이드

- 구글이 만든 모바일 운영체제
- 리눅스 기반 환경

## 코틀린

- 자바처럼 JVM 에서 동작하는 프로그래밍 언어

### 코틀린의 특징

- 간결함
- 더 안전한 코드 : NPE 와 같은 일반적인 프로그래밍 실수를 방지하는 다양한 장치 제공
- 호환성 : 자바와 100% 호환
- 구조화된 동시 실행 : 코루틴을 사용하면 비동기 코드를 쉽게 사용할 수 있음
    - 네트워크 호출, 로컬 데이터 액세스에 이르는 백그라운드 작업 등

## app 폴더 구조

### manifests

- AndroidManifest.xml 파일 하나만 존재
- 앱과 관련된 기본적인 설정
    - 앱의 패키지 이름, 앱 구성요소, 권한 정의
        - 휴대폰의 위치 정보, 전화번호부 등의 민감한 정보에 접근하는 데 필요한 권한 설정
- 이전에는 앱의 패키지 이름이 AndroidManifest.xml 에 있었으나 최근 버전에는 build.gradle 로 이동
- application 태그 : 액티비티, 서비스, 콘텐츠 제공자, 브로드캐스트 리시버 등 정의
- activity 태그 : 기본 생성된 MainActivity 정의
    - 액티비티 : 앱에서 눈에 보이는 화면
    - 추가적으로 액티비티를 생성하는 경우 해당 파일에 액티비티로 정의

### kotlin+java

- 앱의 로직을 담당하는 부분

### res (resource)

- 앱에서 사용하는 이미지, 레이아웃, 색상 값과 같은 자원을 모아놓은 곳
- drowable : 앱에서 사용할 이미지, 아이콘 파일
- layout : 액티비티나 프래그먼트와 같이 화면에 보여줄 구성요소들의 레이아웃 (xml)
- mipmap : 런처에 등록할 이미지
    - 런처 : 앱을 실행할 때 누르는 아이콘
- values : 색상이나 문자열과 같은 설정 파일 (xml)
- xml

## Gradle Scripts 폴더 구조

- Gradle : 소스 코드를 컴파일하고 apk 파일로 패키징하는 빌드 과정을 자동화 (빌드 시스템)
- 모듈 : 하나의 프로젝트를 구성하는 단위
- 프로젝트 : 여러 모듈을 묶어 하나로 관리하는 개념

### build.gradle (Project: {모듈이름})

- 프로젝트 수준에서 모든 모듈에 영향을 미치는 설정

### build.gradle (Module: {모듈이름})

- 해당 모듈에만 영향을 미치는 설정

## 변수와 상수

- var : 변수 / val : 상수
- var 타입이더라도 한 번 타입이 지정되면(추론되면) 다른 타입의 데이터로 수정할 수 없음
    - 코틀린은 자바를 기반으로 만든 언어이기 때문에 자바의 속성을 그대로 따라감
- var 타입으로 동일한 이름의 변수를 재정의(수정X) 할 수 없음

```kotlin
val pi : Double = 3.14
println(pi)

val pi2 = 3.14 // 타입 추론
println(pi2)

var age = 21
println(age)
age = 25 // var 는 변수이므로 수정 가능
println(age)
```

## 자료형

- **코틀린의 자료형은 모두 참조형**
- 코틀린의 참조형 클래스 이름은 자바의 기본 자료형에서 클래스 형식으로 변경한 것과 동일

### 정수 자료형

| 자료형 | 크기 |
| --- | --- |
| Byte | 1 Byte |
| Short | 2 Byte |
| Int | 4 Byte |
| Long | 8 Byte |
- 코틀린에서 정수의 경우 타입 추론 시 Int 타입으로 추론하기 때문에 자료형 지정 필요

### 실수 자료형

| 자료형 | 크기 |
| --- | --- |
| Double | 8 Byte |
| Float | 4 Byte |
- 실수의 경우 자료형을 명시하지 않으면 Double 로 타입 추론

### 문자 자료형

- Char
- String

### 논리 자료형

- Boolean

### 배열 자료형

- Array
- arrayOf : 배열을 만들 수 있는 함수

```kotlin
val stringArray : Array<String> = arrayOf("apple", "banana", "grape")
val intArray : Array<Int> = arrayOf(1, 2, 3)

println(stringArray[0]) // apple
println(intArray[2]) // 3
```

### 명시적 형변환

- 변환될 자료형을 직접 지정하는 것

```kotlin
val score = 100
val scoreString = score.toString()
val scoreDouble = score.toDouble()

print(scoreDouble) // 100.0##scratch##generated##kotlin.Unit
```

## 함수

```kotlin
fun 함수명 (매개변수) : 반환자료형 {
	// 실행할 코드
}
```

- Unit : 자바의 void (생략가능)

    ```kotlin
    fun printAge(age : Int) : Unit {
        println(age)
    }
    printAge(15) // 15
    ```

- 반환 자료형이 Unit 이거나 단일 표현식 함수만 반환 자료형 생략 가능

    ```kotlin
    fun addNum(a : Int, b : Int) : Int {
        return a + b
    }
    println(addNum(2, 3)) // 5
    
    // Type mismatch: inferred type is Int but Unit was expected
    fun addNum2(a : Int, b : Int) {
        return a + b
    }
    println(addNum2(2, 3))
    
    fun minusNum(a : Int, b : Int) = a - b
    println(minusNum(minusNum(1000, 200), 100)) // 700
    ```


## 문자열 템플릿

```kotlin
val price = 3000
val tax = 300

val originalPrice = "The original price is ${price}"
val totalPrice = "The total price is ${price + tax}"

println(originalPrice) // The original price is 3000
println(totalPrice) // The total price is 3300
```

## 제어문

### 범위 클래스

- IntRagne, LongRange, CharRange

```kotlin
val numRange : IntRange = 1..5

println(numRange.contains(3)) // true
println(numRange.contains(10)) // false

val charRange : CharRange = 'a'..'e'

println(charRange.contains('b')) // true
println(charRange.contains('z')) // false
```

### for 문

- for 문 사용 시 내부적으로 범위 클래스를 주로 사용함
- `1..5` 처럼 범위 클래스 사용 시 IntRange 객체가 생성되어 해당 객체를 순회
- array.withIndex() 함수는 python 의 enumerate() 함수와 같이 현재 배열의 값과 인덱스 여부를 함께 반환

```kotlin
// 1 부터 5 까지 순회
for (i in 1..5) {
    println(i)
}

// 5 를 시작으로 1 씩 적은 값 순회
for (i in 5 downTo 1) {
    println(i)
}

// 1 부터 10 까지 2칸씩 순회
for (i in 1..10 step 2) {
    println(i)
}

val students = arrayOf("jun-gi", "jun-su")

for (name in students) {
    println(name)
}

// 배열의 인덱스 포함
for ((index, name2) in students.withIndex()) {
    println("Index : $index Name : $name2")
}
```

### while 문

```kotlin
var num = 1
while (num < 5) {
    println(num)
    num++
}

var num2 = 1
do {
    num2++
    println(num2)
} while (num2 < 5)
```

### if 문

- 코틀린의 if 는 문(statement) 이 아닌 식(expression)
    - `statement` : 코드에서 실행될 수 있는 독립적인 코드 조각
    - `expression` : 값으로 표현되는 것
- if 는 명령문이 아닌 값 자체이기 때문에 return 으로 바로 값을 반환할 수 있음
- 또한 표현식으로 간단하게 나타낼 수 있기 때문에 삼항연산자는 없음

```kotlin
val examScore : Int = 60
var isPass : Boolean = false

if (examScore > 80) {
    isPass = true
}

println("시험 결과 : $isPass") // 시험 결과 : false

val examScore2 : Int = 99
if (examScore2 == 100) {
    println("만점이네요.")
} else {
    println("만점은 아니에요.")
}
// 만점은 아니에요.

val myAge = 40
val isAdult = if (myAge > 30) true else false

println("성인 여부 : $isAdult") // 성인 여부 : true
```

### when 문

- java 의 switch 문을 대체할 수 있는 표현식
- switch 문 처럼 상위 조건을 먼저 체크 후 하위 조건으로 넘어감
    - 범위가 겹칠 경우 상위 조건으로 처리됨
- if 와 마찬가지로 when 도 표현식이기 때문에 값을 반환하는 형식으로 사용할 수 있음

```kotlin
val weather : Int = 15

when (weather) {
    -20 -> { println("매우 추운 날씨") } // 값 하나
    11, 12, 13, 14 -> { println("쌀쌀한 날씨") } // 값 여러 개
    in 15..26 -> { println("활동하기 좋은 날씨") } // 범위 안에 들어가는 경우
    
    !in -30..50 -> { println("잘못된 값") } // 범위 안에 들어가지 않는 경우
    else -> { println("잘 모르는 값") } // 위 경우가 모두 아닐 때
}
// 활동하기 좋은 날씨

val essayScore : Int = 95
val grade = when (essayScore) {
    in 0..40 -> "D"
    in 41..70 -> "C"
    in 71..90 -> "B"
    else -> "A"
}
println("에세이 학점 : $grade") // 에세이 학점 : A
```

## 컬렉션 (Collection)

- java 의 list, set, map 을 포함하여 편리한 함수 추가 제공
- 코틀린은 컬렉션을 읽기 전용(immutable) 컬렉션과 읽기-쓰기(mutable) 컬렉션으로 구분됨

### 리스트 (List)

```kotlin
val numImmutableList = listOf(1, 2, 3)
// numImmutableList[0] = 1 // 에러 발생

val numMutableList = mutableListOf(1, 2, 3)
numMutableList[0] = 100

println(numMutableList) // [100, 2, 3]
println(numMutableList[0]) // 100

println(numMutableList.contains(1)) // false
println(numMutableList.contains(2)) // true
```

### 셋 (Set)

```kotlin
val immutableSet = setOf(1, 1, 2, 2, 2, 3, 3)
println(immutableSet) // [1, 2, 3]

val mutableSet = mutableSetOf(1, 2, 3, 3, 3, 3)
mutableSet.add(100) // true
mutableSet.remove(1) // true
mutableSet.remove(200) // false

println(mutableSet) // [2, 3, 100]
println(mutableSet.contains(1)) // false
```

### 맵 (Map)

```kotlin
val immutableMap = mapOf(
    "name" to "junsu",
    "age" to 13,
    "age" to 15,
    "height" to 160)
println(immutableMap) // {name=junsu, age=15, height=160}

val mutableMap = mutableMapOf(
    Pair("돈까스", "일식"),
    Pair("짜장면", "중식"),
    Pair("김치", "중식")
)
mutableMap.put("막국수", "한식") // null
mutableMap.remove("돈까스") // 일식
mutableMap.replace("김치", "한식") // 중식, put 으로 사용해도 결과는 동일
println(mutableMap) // {짜장면=중식, 김치=한식, 막국수=한식}
```

