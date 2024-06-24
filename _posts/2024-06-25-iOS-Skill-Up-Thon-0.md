---
layout: post
title: iOS SkillUpThon 0
subtitle: iOS SkillUpThon 0
author: 민욱 최 
excerpt_image: ../assets/images/iOS_banner.jpg
categories: iOS
banner:
  image: ../assets/images/iOS_banner.jpg
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 3.5em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: iOS Swift 
---
       
## 화면 전환 방법 
---
1. present
2. NavigationController + Source (push)
3. NavigationController + Segue

### 화면 전환 방법1: present
* ViewController가 다른 ViewController 호출 (present)
* 다른 ViewController를 Modal로 띄움
* UIViewController에 정의된 present메소드를 사용
* 돌아올 때
    * presentingViewController?.dismiss(animated:)
* full screen으로 화면 띄우기
    * vc.modalPresentationStyle = .fullScreen

## Array
---
> 연결리스트 (linked list) 

### Swift의 배열은 generic 구조체

``` Swift
var x : [Int] = [] // 빈 배열 
```

### Array의 자료형

``` Swift
let number = [1,2,3,4] 
print(type(of:number)) // Array<Int>
```

### 빈 배열 주의사항

``` Swift
let number : [Int] = []
// 빈 배열을 let으로 만들 수는 있지만 초기 값에서 변경 불가이니 배열의 의미 없음
// 따라서 var로 수정해야함
```

``` Swift
var number : [Int] = []

//number[0] = 1 // crash 방을 만든 후 사용

number.append(1)
print(number)
number[0] = 10
print(number) // 10
``` 

### Array(repeating:count:)

* 특정 값으로 원하는 개수만큼 초기화

``` Swift
var x1 = Array(repeating: 0, count: 5)
```

### for ~ in 으로 배열 항목 접근

``` Swift
let color = ["red", "green", "blue"]
print(colors)

for color in colors { 
    print(color) // red green blue
}
```

### 항목이 몇 개인지, 비어있는지 알아내기

``` Swift
let num = [1,2,3,4]
var x = [Int]()
print(num.isEmpty) // 배열이 비어있나? => false
print(x.isEmpty)

if num.isEmpty {
    print("비어 있습니다.")
}
else {
    print(num.count)// 배열 항목의 개수 4
}
```

### first와 last 프로퍼티

``` Swift
let num = [1,2,3,4]

print(num.first, num.last) // Optional(1) Optional(4)
if let f = num.first, let l = num.last {
    print(f,l) // 1,4
}
// Optional로 나오기 때문에 Optional Binding을 통해 unwrapping 해야한다.
```

### Array: 추가/제거

``` Swift
var num = [1,2,3]
num.append(4)
num.append(contentOf: [6,7,8])
num.insert(5, at:4)
print(num) // [1,2,3,4,5,6,7,8]
num.remove(at:3)
print(num) // [1,2,3,5,6,7,8]
```

### Array는 구조체이므로 값 타입

``` Swift
var num = [1,2,3]
var x = num
num[0] = 100
print(num) // 100,2,3
print(x) // 100,2,3
```

### Array 요소의 최댓값 최소값 : max(), min()

``` Swift
var num = [1,2,3,10,20]
print(num) // 1 2 3 10 20
print(num.min()!) // 1
print(num.max()!) // 20
//Optional unwrapping 해줘야함
```

### Array 요소의 정렬

``` Swift
var num = [1,5,3,2,4]
num.sort() // 오름차순 정렬, 원본 변경
num.sort(by:>) // 내림차순 정렬, 원본 변경
num.reverse() // 반대로 정렬, 원본 변경

print(num.sorted()) // 오름차순 정렬, 원본 그대로
print(num.sorted(by:>)) // 내림차순 정렬, 원본 그대로
```

## Dictionary
---

> Generic 구조체 Dictionary

### Hashable 프로토콜

> 정수 Hash 값을 제공하는 타입

### Dictionary 제약 조건

* 데이터를 키~값 쌍의 형태로 저장하고 관리
* 배열과 비슷한 목적의 작업을 하지만 순서가 없음
* Dictionary에 저장된 각 항목은 값을 참조하고 접근하는 데 사용되는 유일한 키와 연결되어있음
* Dictionary의 키는 해시 가능한 타입이어야 함
* Swift의 기본 타입(String, Int, Double, Bool 등)은 해쉬 가능하므로 Dictionary의 키로 사용할 수 있음
* Optional, Array, Set도 키로 가능


### Swift의 Dictionary는 generic 구조체

``` Swift
var x = [Int:String]()

var a1 : [Int:String] = [1:"일", 2:"이", 3:"삼", 4:"사"]
var colors : [String:String] = ["빨강":"red", "초록":"green", "파랑":"blue"]
var colors1 : Dictionary<String,String>  = ["빨강":"red", "초록":"green", "파랑":"blue"]
print(type(of:colors)) // Dictionary<String, String>
```

### Dictionary 항목 접근/변경/삭제

``` Swift
var number : [Int:String] = [1:"일", 2:"이", 3:"삼"]
print(number[1]!, number[2]!, number[3]!) // Optional unwrapping 
// Array가 아니라 key가 Int형인 Dictionary
number.updateValue("둘", forKey:2) // 
print(number) // [1:"일", 2:"둘", 3:"삼"]
number.removeValue(forKey:1) // 1이라는 key를 갖는 value를 삭제

```

### Keys나 Value형을 Array형으로 만들기

``` Swift
var colors = ["빨강":"red", "초록":"green", "파랑":"blue"]
var kColors = colors.keys
print(type(of:kColors)) // Keys String Array가 아님

var kColors1 = [String](colors.keys) // Array의 init으로 String Array로 변경

```

## Swift 고차함수(Higher-order function)
---

* 다른 함수를 매개 변수로 받거나 함수 실행 결과를 함수로 반환하는 함수
* 스위프트의 함수(클로저)는 1급 객체(first class object) 또는 1급 시민(first class citizen)이기 때문에 함수의 매개변수로 전달할 수 있으며, 함수의 리턴 값으로 반환할 수 있음
* Swift의 대표적인 고차함수
    * map
    * filter
    * reduce

## map
---

* 컨테이너가 담고 있는 각각의 값을 매개변수를 통해 받은 함수에 적용한 후 새로운 컨테이너를 생성하여 반환
    * 기존 컨테이너의 값은 변경되지 않음
* 시퀀스(Sequence), 콜렉션(Collection) 프로토콜을 따르는 타입과 옵셔널은 맵 사용할 수 있음
    * Array, Dictionary, Set, Optional
* for-in 구문을 사용하는 것과 비슷하나 코드가 더 간결

``` Swift
let arr = [0,1,2,3]
let num = arr.map( {(x:Int) -> Int in
    return x + 1
})

print(arr) // [0,1,2,3]
print(num) // [1,2,3,4]

var arr1 = arr.map {$0+1}
print(arr1) // [1,2,3,4]
print(arr) // [0,1,2,3]
```



<br>
<br>
<br>
<br>
<br>
<br>
<br>

## 출처
---
* Smile Han의 iOS 프로그래밍 실무, 한성현(출판 예정), PPT로 제공  
