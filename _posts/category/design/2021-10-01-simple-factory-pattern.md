---
layout: post
title: Factory Pattern
description: >
    Factory Pattern
sitemap: false
hide_last_modified: false
categories:
  - category
  - design-pattern
---


# Factory Pattern

* toc
{:toc .large-only}

## OOP에서 팩토리란?

- 다른 클래스의 객체를 생성하는 클래스를 말한다.

## 팩토리 패턴이란?

> **팩토리 패턴에서는 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 객체를 만들지는 서브 클래스에서 결정하게 만드는 패턴이다.**
> 

## 팩토리 패턴이 필요한 이유?

- 객체 생성하는 코드를 분리하여 클라이언트 코드와 결합도(의존성)을 낮추어 코드를 건드리는 횟수를 최소화 할 수 있다.
    - 클라이언트 → 객체 생성 코드를 호출하는 쪽
    - 객체를 생성하는 코드를 분리 하였기 때문에 객체의 추가,수정이 일어나도 객체를 생성하는 코드만 건들면 된다.
- 객체를 생성하는 코드를 추상화하여 코드를 한곳에서 관리하지 않으면, 변화(생성,수정,삭제)가 발생 했을 때 해당 클라이언트 코드를 전부 수정해줘야 한다. (예를 들어 클래스 `__init__` 에 인자 추가) 즉, 객체지향 디자인패턴 원칙 <개방폐쇄의 원칙> 확장에 대해서는 열려고있고 변화에 대해서는 닫혀있어야 한다. 때문에 변화가 일어날 수 있는 객체 생성 담당하는 클래스를 만들어 한곳에서 관리하여 결합도를 줄이기 위해 사용한다.

## 현실에서 팩토리 패턴의 예

- 장난감을 제조하는 공장이 있다.
- 항상 자동차 장난감만 만들어왔지만 인형에 대한 시장의 수요가 늘어나는 상황을 고려해 CEO는 급히 인형도 제조하기로 했다.
- 이 경우 공장의 제조 기계는 인터페이스이고 CEO는 클라이언트다.
- CEO는 제조하려는 객체(완제품)과 제품을 만드는 인터페이스(생산 기계)에 대해서만 알고 있다.
- 실제로 객체(완제품)을 만드는 건 인터페이스(생산 기계)가 내부적으로 여러 공정을 거쳐서 만들게 된다.

## 팩토리 패턴의 유형

### 심플 팩토리 패턴

- 인터페이스는 객체 생성 로직을 숨기고 객체를 생성한다.

### 팩토리 메소드 패턴

- 인터페이스를 통해 객체를 생성하지만 서브 클래스가 객체 생성에 필요한 클래스를 선택한다.

### 추상 팩토리 패턴

- 추상 팩토리는 객체 생성에 필요한 클래스를 노출하지 않고 객체를 생성하는 인터페이스다. 내부적으로 다른 팩토리 객체를 생성한다.

## 심플 팩토리 패턴 
***인터페이스는 객체 생성 로직을 숨기고 객체를 생성한다.***

- 심플 팩토리 패턴은 하나의 패턴으로 인정하지 않기도 해서 팩토리 메소드 패턴과 추상 팩토리 패턴을 이해하기 위한 개념 정도로 생각하면 된다.
- 팩토리를 사용하면 여러 종류의 객체를 사용자가 직접 클래스를 호출하지 않고 생성할 수 있다.

```python
from abc import ABCMeta, abstractmethod

class Animal(metaclass=ABCMeta):
    @abstractmethod
    def do_say(self):
        pass

class Dog(Animal):
    def do_say(self):
        print("왈왈!!")

class Cat(Animal):
    def do_say(self):
        print("냐옹~~")

# forest factory 정의
class ForestFactory:
    def make_sound(self, object_type):
        return eval(object_type)().do_say()

# 클라이언트 코드
if __name__ == "__main__":
    forest_factory = ForestFactory()
    animal = input("Which animal should make_sound Dog or Cat? ")
    forest_factory.make_sound(animal)

-----------
result
-----------
Which animal should make_sound Dog or Cat? Cat
냐옹~~
```

- Animal은 추상 클래스이고 `do_say()` 메소드를 포함한다.
    - `ABCMeta`는 파이썬에서 특정 클래스를 Abstract로 선언하는 특수 메타클래스이다.