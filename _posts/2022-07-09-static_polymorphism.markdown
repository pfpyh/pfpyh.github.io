---
layout: post
title:  "Static Polymorphism"
date:   2022-07-09 17:00:00 +0900
categories: C++ TMP
---
_아래 내용은 알고 있는 내용을 정리하는 차원에서 적은 글입니다._  
  
C++를 사용하면서 상속을 구현할 때 virtual keyword를 사용해왔다. (Dynamic polymorphism)  
virtual은 runtime에 binding이 이루어지기 때문에 정말 하드하게 성능을 생각한다면 polymorphism을 포기하는 것이 좋다고 알고 있었다.  
하지만 CRTP를 알고나서 이를 이용해서 static polymorphism을 구현할 수 있고 이를 통해 dynamic 사용 시 발생하는 overhead를 없앨 수 있다는 것을 알게 되었다.  
  
*CRTP (Curiously recurring template pattern)*  
> 어떤 class X가 X 자신을 template 인자로 받는 pattern  
  
<script src="https://gist.github.com/pfpyh/2486803d6b6bae7019d7f264375dbec9.js"></script>  
  
*Static polymorphism*  
> CRTP를 이용해서 Base class를 구현하고 Base를 Derived로 casting해서 실제 구현부를 호출하도록 함으로써 이루어진다.  
  
<script src="https://gist.github.com/pfpyh/e6e4b546c23ac5c6962c54d5a0f0cac7.js"></script>  
  
[장점]  
Runtime에 binding이 발생하지 않기 때문에 virtual에 비해 overhead가 적다.  
- virtual의 경우 50% 정도의 overhead가 존재한다고 함.  
- 최근 예측 알고리즘의 발전으로 실제로는 5~15% 정도의 overhead가 있다고 함. (Platform마다 다름)  
  
Compile time에 data형 검사가 발생하기 때문에 일반적으로 dynamic보다 안전하다.  
- 예를 들어 잘못된 instance type을 넣는 등의 문제를 compile time에 미리 알수 있다.  
  
[단점]  
단점은 TMP의 단점처럼 잘못 설계할 경우 binary size가 증가하는 등의 문제가 있을 수 있다.  
  
  
아래는 static polymophism을 적용해서 구현해본 thread 처리를 위한 framework.  
<script src="https://gist.github.com/pfpyh/cc8147d1b921c08bc9e63166e3477e0d.js"></script>  