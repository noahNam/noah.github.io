---
layout: post
title: Singleton pattern
description: >
    Singleton pattern
sitemap: false
hide_last_modified: false
categories:
  - category
  - design_pattern
---


# 싱글톤 패턴

* toc
{:toc .large-only}

## 싱글톤이란?

- 글로벌하게 접근 가능한 하나의 객체를 제공하는 패턴이다.

## 싱글톤 목적

- 클래스에 대한 단일 객체 생성
- 전역 객체 제공
- 공유된 리소스에 대한 동시 접근 제어

## 언제 쓰이나?

- 싱글톤은 로깅이나 데이터베이스 관련 작업과 같이 동일한 리소스에 대한 동시 요청의 충돌을 방지하기 위해 하나의 인스턴스를 공유할 때 사용된다.
- 예를 들어 데이터의 일관성 유지를 위해 DB에 작업을 수행하는 하나의 DB 객체는 하나가 필요하다. 또한, 여러 서비스의 로그를 한 개의 로그 파일에 순차적으로 쌓기 위해서는 하나의 로깅 객체를 사용한다.


## 싱글톤 패턴의 특징

- 어플리케이션을 개발할 때 스레드 풀과 캐시, 대화 상자, 레지스트리 설정 등 한 개의 객체만 필요한 경우가 많은데 이러한 경우에 여러 개의 객체를 생성하는 것은 리소스 낭비이고, 싱글톤 패턴이 유리하다.
- 싱글톤은 글로벌 액세스 지점을 제공하는, 단점이 거의 없는 검증된 패턴이다.

## 싱글톤 패턴의 단점

- 전역 변수의 값이 변경되었을 때 사이드 이펙트가 매우 클 수 있다.
- 하나의 객체만을 생성하기 때문에 같은 객체를 참조하는 여러개의 참조자가 생긴다.
    - a = A()
    - b = A()
    - c = A()

## 단일 객체 생성 방법

- 생성자를 private으로 선언하고 객체를 초기화하는 static 함수를 만들면 간단하게 싱글톤을 구현할 수 있다. 즉, 첫 호출에 객체가 생성되고 클래스는 동일한 객체를 계속 반환한다.
- 하지만 파이썬에서는 생성자를 private으로 선언할 수 없기 때문에 다른 방법이 필요하다.

### 파이썬 싱글톤 패턴 구현

1. 하나의 싱글톤 클래스 인스턴스를 생성한다.
2. 이미 생성된 인스턴스가 있다면 재사용한다.

```python
class Singleton:
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, "instance"):
            print("__new__ is called\n")
            cls.instance = super().__new__(cls)
        return cls.instance

    def __init__(self):
        print("__init__ is called\n")

s = Singleton()
print(f"Object created {s}")

s1 = Singleton()
print(f"Object created {s1}")

-----------
result
-----------
__new__ is called
__init__ is called
Object created <__main__.Singleton object at 0x7fdb6970d6d0>

__init__ is called
Object created <__main__.Singleton object at 0x7fdb6970d6d0>
```
- 위 코드를 보면 `__new__` 함수 (파이썬 전용 특수 생성자) 를 오버라이드해 객체를 생성한다.
    - `__new__`는 클래스에 정의되어 있지 않으면 알아서 object 클래스의 `__new__`가 호출되어 객체가 생성된다. 생성된 객체에 속성(property)을 추가할 때 `__init__` 가 호출되는 것이다.
    - 클래스를 정의할 때 `__new__` 메서드를 작성할 수도 있는데 이 경우 object 클래스의 `__new__`가 아니라 클래스의 정의된 `__new__`가  오버라이드(override) 되어 호출된다. 사용자가 어떤 목적으로 클래스의 생성 과정에 관여하고 싶을 때 직접 `__new__` 메서드를 클래스의 추가함으로써 object 클래스의 `__new__`가 아니라 사용자가 정의한 `__new__` 메서드가 호출되도록 한다.

- `__new__` 함수는 s 객체가 이미 존재하는지 확인하고 `hasattr` 함수(해당 객체가 명시한 속성을 가지고 있는지 확인하는 파이썬 함수) 는 `cls` 객체가 `instance` 속성을 가지고 있는지 확인한다. 즉, 클래스 객체가 이미 존재하는지 확인하는 과정이다.
- s1 객체를 요청하면 `hasattr()`은 이미 객체가 생성됐음을 확인하고 해당 인스턴스 (0x102078ba8)를 반환한다.
- 하지만 위 코드에는 문제가 있다. 객체는 유일한 객체가 생성 되었지만 생성된 객체를 초기화하는 `__init__`은 여전히 여러번 호출되고 있기 때문이다. 이 경우 `__init__` 호출 시점에 따라 서로 다른 값으로 초기화를 하는 경우 객체의 속성 값이 달라지는 문제가 발생할 수 있다. 이 문제를 해결하기 위해 아래 코드를 추가 해준다.

```python
class Singleton:
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, "instance"):
            print("__new__ is called\n")
            cls.instance = super().__new__(cls)
        return cls.instance

    def __init__(self, data): # 호출 시점에 따른 객체 속성 값 변경을 보기 위해 data param을 추가
      cls = type(self)   # self 객체의 class를 가져온다.
      if not hasattr(cls, "_init"): # 클래스 객체에 _init 속성이 없다면
          print("__init__ is called\n")
          self.data = data
          cls._init = True

s = Singleton(3)
print(f"Object created {s}")
print(s.data)

s1 = Singleton(4)
print(f"Object created {s1}")
print(s1.data)

-----------
result
-----------
__new__ is called
__init__ is called

Object created <__main__.Singleton object at 0x7febd53f26d0>
3
Object created <__main__.Singleton object at 0x7febd53f26d0>
3
```
- 위에서 문제가 되었던 `__init__` 함수가 생성 초기에 한번만 실행이 되어 속성값 초기화가 이루어지지 않는 것을 확인할 수 있다.

## 게으른 초기화(Lazy instantitation)

### 게으른 초기화란?

- 싱글톤 패턴을 기반으로 하는 초기화 방식이다.
- 모듈을 임포트할 때 아직 필요하지 않은 시점에 실수로 객체를 미리 생성하는 경우가 있는데, 게으른 초기화는 인스턴스를 꼭 필요할 때 생성한다.
- 따라서, 제한된 리소스를 관리하는 경우 효율적이다.

### 게으른 초기화 구현
```python
class Singleton:
    __isinstance = None

    # def __new__(cls, *args, **kwargs):
    #     if not hasattr(cls, "instance"):
    #         print("__new__ is called\n")
    #         cls.instance = super().__new__(cls)
    #     return cls.instance

    def __init__(self):
        if not Singleton.__isinstance:
            print("__init__ method called")
        else:
            print(f"instance already created: {self.get_instance()}")

    @classmethod
    def get_instance(cls):
        if not cls.__isinstance:
            cls.__isinstance = Singleton()
        return cls.__isinstance

s = Singleton()  # 클래스를 초기화했지만 객체는 생성하지 않음
print("------------------------")
print(f"Object created {Singleton.get_instance()}")  # 객체 생성

print("------------------------")
s1 = Singleton()  # 객체는 이미 생성되었음

-----------
result
-----------
__init__ method called
------------------------
__init__ method called
Object created <__main__.Singleton object at 0x7fb61d665130>
------------------------
instance already created: <__main__.Singleton object at 0x7fb61d665130>
```
- ~~위 코드의 `s = Singleton()` 는 `__init__` 함수를 호출하지만 객체는 생성하지 않고,~~  `Singleton.get_instance()` 호출 시점에 객체가 생성된다.
- 굳이 클래스의 생성 과정에 관여하지 않으므로 object 클래스의 `__new__` 를 사용하고, 오버라이드 하지 않는다.

## 모듈 싱클톤

- 파이썬의 import 방식으로 인해 모든 모듈은 기본적으로 싱글톤이다.

### 모듈 동작 방식

1. 파이썬 모듈이 import 됐는지 확인
2. 이미 import 된 경우, 해당 모듈의 객체를 반환하고 import 되지 않은 겨우, import 후 초기화한다.
3. 모듈은 import와 동시에 초기화된다. 모듈을 다시 import 하면 초기화 되지 않고 하나의 객체를 유지 및 반환한다.

## 모노스테이트 싱글톤 패턴

- **알렉스 마르텔리(Python Software Foundation의 연구원)** 는 객체의 생성 여부보다는 객체의 상태와 행위가 더 중요하다고 주장하였고, 모노스테이트 싱글톤 패턴을 소개하였다.
- 이름 그대로 모든 객체가 같은 상태를 공유하는 패턴이다. 즉, 인스턴스를 많이 생성 하더라도 결국 하나의 상태 값을 서로 공유 한다.

```python
class MonoStateSingleton:
    __shared_state = {}

    def __init__(self):
        self.x = 30
        self.y = 40
        self.__dict__ = self.__shared_state
        pass

m1 = MonoStateSingleton()
m2 = MonoStateSingleton()
m1.x = 99
m1.y = 999
m1.z = 9999

print(f"MonoStateSingleton Object 'm1' : {m1}")  # m1과 m2는 다른 객체이다.
print(f"MonoStateSingleton Object 'm2' : {m2}")
print(f"Object State 'm1' : {m1.__dict__}")  # m1과 m2는 상태를 공유한다.
print(f"Object State 'm1' : {m2.__dict__}")

-----------
result
-----------
MonoStateSingleton Object 'm1' : <__main__.MonoStateSingleton object at 0x7f9797b02f70>
MonoStateSingleton Object 'm2' : <__main__.MonoStateSingleton object at 0x7f9797b02e80>
Object State 'm1' : {'x': 99, 'y': 999, 'z': 9999}
Object State 'm1' : {'x': 99, 'y': 999, 'z': 9999}
```

- `__dict__` 특수 속성을 `__shared_state` 클래스 속성으로 지정했다.
- 파이썬은 `__dict__` 속성에 클래스 내 모든 객체의 상태를 저장한다. 위 코드에서는 모든 인스턴스의 `__dict__` 를 같은 `__shared_state` 지정했다.
- 하나의 객체만 생성하는 기존의 싱글톤과 달리 m1과 m2 인스턴스를 초기화하면 두 개의 객체가 생성된다. 하지만 `m1.__dict__` 와 `m2.__dict__` 는 같다.
- 따라서 m1 객체의 x 값을 99로 설정하면 모든 객체가 공유하는 `__dict__` 변수에 복사되어 m2의 x값도 99가 된다.

### 모노스테이트 싱글톤에 대한 개인적 생각

- 클래스를 생성할 때마다 초기화 되는 경우 기존의 상태값을 모르기 때문에 상태를 기억하여 공유해야 할 때는 적절하다.
- 하지만 싱글톤 사용의 대전제가 하나의 객체를 사용함으로써 동시 요청의 충돌을 방지하고 리소스를 효율적으로 관리하기 위함인데 이러한 대전제에 어긋나는 것 같아서 과연 싱글톤이라고 불러야 하는지 의문이 든다. 물론 객체의 단일성 측면으로만 본다면 싱글톤이라고 부를 수 있을 것같다.
- 아직까지는 어느경우에 써야 효율적인지 판단이 서질 않는다.
- 다만, 리서칭 결과 모노스테이트를 사용할때 일반 클래스와 차이점이 없기 때문에 일반 클래스 사용과 동일한 기준에서 사용하면 될 것으로 보인다.