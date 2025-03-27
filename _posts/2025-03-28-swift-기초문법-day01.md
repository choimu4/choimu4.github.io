---
layout: post
title: Swift 기초문법 day01
subtitle: Swift 기초문법 day01 - 조건문
author: 민욱 최 
excerpt_image: ../assets/images/swift_banner.PNG
categories: Swift
banner:
  image: ../assets/images/swift_banner.PNG
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 3.5em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: iOS Swift
---
      
## 머리말  
---  
본 게시물은 https://youtu.be/EXtpt5Skzck?si=N3oXtUEjTZDvGB9h 을 기반으로 제작하였습니다.


시작.


## 조건문
---

``` swift
var isDarkMode : Bool = true

if (isDarkMode == true) {
  print("다크모드 입니다.")
}
```
기본적인 조건문 형태   

<br>

``` swift
var isDarkMode : Bool true

if (isDarkMode != true) {
    print("다크모드가 아닙니다.")
} else {
    print("다크모드 입니다.")
}
```
!= 연산자를 활용하는 방법

<br>   

``` swift
if isDarkMode == true {
    print("다크모드 입니다.")
}
```
다음과 같이 괄호를 생략할 수 있다.

<br>

``` swift
if isDarkMode {
    print("다크모드 입니다.")
}

if !isDarkMode {
    print("다크모드가 아닙니다.")
} else {
    print("다크모드 입니다.")
}
```
다음과 같이 연산자를 생략하는 방법도 있다.

<br>

``` swift
var title : String = isDarkMode ? "다크모드 입니다." : "다크모드가 아닙니다."
print("title: \(title)")
```
삼항연산자를 활용하는 방법


## 마무리

마침.s