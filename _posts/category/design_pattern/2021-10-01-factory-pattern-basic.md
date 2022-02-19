---
layout: post
title: Singleton metaclass
description: >
    Singleton metaclass
sitemap: false
hide_last_modified: false
categories:
  - category
  - design_pattern
---


# 싱글톤과 메타클래스

* toc
{:toc .large-only}

## 싱글톤과 메타클래스

### Metaclass

- 메타클래스는 클래스의 클래스다. 즉 클래스는 자신의 메타클래스의 인스턴스라고 할 수 있다.
    - a=1, type(a) → <type ‘int’> 반환
        - a는 int형 변수이기 때문에
    - type(int) → <type ‘type’> 반환
        - int의 메타클래스는 type 클래스라는 의미로 보면 된다.
- 클래스는 메타클래스가 정의한다. 클래스 A의 객체를 초기화하면 파이썬은 내부적으로 A = Metakls(name, bases, dict)를 수행한다.
    - Metakls: 메타클래스
    - name: 클래스명
    - bases: 기본클래스
    - dict: 클래스 속성
    
- 메타클래스를 사용하면 이미 정의된 파이썬 클래스를 통해 새로운 형식의 클래스를 생성할 수 있다. 말로 설명하긴 어려우니 아래 코드를 작성해 두었다.

```python
class MyInt(type):
    def __call__(cls, *args, **kwargs):
        print("**** Here's My int *****", args)
        print("이제 이 개체로 원하는 모든 작업을 수행하십시오")
        return type.__call__(cls,  *args, **kwargs)

class int(metaclass=MyInt):
    def __init__(self, x, y):
        self.x = x
        self.y = y

i = int(4, 5)
print(type(int)) # type이 메타클래스가 아닌 MyInt가 메타클래스가 되었다.

---------------
result
---------------
**** Here's My int ***** (4, 5)
이제 이 개체로 원하는 모든 작업을 수행하십시오
<class '__main__.MyInt'>
```

- `int` 클래스의 메타클래스인 `type` 클래스를 상속받아 `MyInt` 클래스를 생성하고 재정의한 `int` 클래스의 metaclass로 `MyInt` 클래스를 설정하였다. 그 결과 흔히 알던 `int` 클래스가 재정의 된 것을 볼 수 있다.
    - `__call__` 메소드는 이미 존재하는 클래스의 객체를 호출할 때 사용되는 특수 메소드다.
    - 위의 코드에서는 `int(4,5)`로 int 클래스를 생성하면 `MyInt` 메타클래스의 `__call__` 이 호출된다.
    - 객체 생성을 메타클래스가 제어한다는 의미이다.

## 그렇다면 싱글톤과 메타클래스는 무슨 연관이 있는가?

- 메타클래스는 클래스 생성과 객체 초기화를 더 세부적으로 제어할 수 있기 때문에, 싱글톤 생성에도 사용할 수 있다.

```python
class MetaSingleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=MetaSingleton):
    pass

Logger1 = Logger()
Logger2 = Logger()
print(Logger1)
print(Logger2)

---------------
result
---------------
<__main__.Logger object at 0x7f8a6650daf0> 
<__main__.Logger object at 0x7f8a6650daf0>
```

```python
class MetaSingleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        print("__call__")
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=MetaSingleton):
    def __init__(self):
        print("__init__ is called\n")

Logger1 = Logger()
Logger2 = Logger()
print(Logger1)
print(Logger2)

---------------
result
---------------
__call__
__init__ is called

__call__
<__main__.Logger object at 0x7f915dc02e80>
<__main__.Logger object at 0x7f915dc02e80>
```