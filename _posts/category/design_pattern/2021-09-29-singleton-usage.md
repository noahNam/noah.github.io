---
layout: post
title: Singleton usage
description: >
    Singleton usage
sitemap: false
hide_last_modified: false
categories:
  - category
  - design_pattern
---


# 싱글톤 사용 사례로 살펴본 장점과 단점

* toc
{:toc .large-only}

## 싱글톤 패턴 사용 사례1

- 클라우드 서비스에서 데이터베이스에 접근하는 하나의 앱 서비스가 있다고 가정한다.
- 기본적으로 데이터베이스는 일관성이 유지되어야 하며, 작업 간 충돌이 발생하지 않아야 한다. 또한, 다수의 DB연산을 처리하려면 메모리와 CPU를 효율적을 사용해야 한다.

```python
import sqlite3

class MetaSingleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        print("__call__")
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=MetaSingleton):
    connection = None

    def connect(self):
        if not self.connection:
            self.connection = sqlite3.connect("db.sqlite3")
            self.cursor = self.connection.cursor()
        return self.cursor

db1 = Database().connect()
db2 = Database().connect()

print(f"Database Objects DB1 : {db1}")
print(f"Database Objects DB1 : {db2}")

---------------
result
---------------
__call__
__call__
Database Objects DB1 : <sqlite3.Cursor object at 0x7ff0278152d0>
Database Objects DB1 : <sqlite3.Cursor object at 0x7ff0278152d0>
```

- `MetaSingleton` 메타클래스가 `__call__` 특수메소드를 사용해 싱글톤 방식으로 객체를 생성한다.
- `Database` 클래스는 `MetaSingleton` 를 메타클래스로 사용하기 때문에 앱에서 DB연산을 요청할 때마다 클래스가 초기화 되지만 내부적으로 하나의 객체만 생성된다.
- 따라서 데이터베이스에 대한 작업은 모두 동기화되고, 빈번한 DB연산 호출에도 메모리와 CPU사용량이 높지 않다.

### 그렇다면 하나의 앱 서비스가 아닌 여러 앱 서비스가 같은 DB에 접속하는 상황에서는?

- 이 경우 각 앱 서비스가 DB에 접근하는 싱글톤을 생성하기 때문에 싱글톤 패턴에 적합하지 않다.
- DB 작업이 동기화 되지 않고 리소스 사용량이 증가한다. 싱글톤 패턴보다는 Connection pooling 을 사용하는 것이 더 효율적이다.

### Connection pooling 특징

- 웹 컨테이너가 실행되면서 Connection(connection) 객체를 미리 Pool(pool)에 생성해 둔다.
- DB와 연결된 Connection(connection)을 미리 생성해서 Pool(pool) 속에 저장해 두고 있다가 필요할 때에 가져다 쓰고 반환한다.
- 미리 생성해두기 때문에 데이터베이스에 부하를 줄이고 유동적으로 연결을 관리 할 수 있습니다.
- Pool 속에 미리 Connection이 생성되어 있기 때문에 Connection을 생성하는 데 드는 연결 시간이 소비되지 않는다.
- Connection을 계속해서 재사용하기 때문에 생성되는 Connection 수가 많지 않다.
- Connection Pool을 사용하면 Connection을 생성하고 닫는 시간이 소모되지 않기 때문에 그만큼 어플리케이션의 실행 속도가 빨라지며, 또한 한 번에 생성될 수 있는 Connection 수를 제어하기 때문에 동시 접속자 수가 몰려도 웹 어플리케이션이 쉽게 다운되지 않는다.

## 싱글톤 패턴 사용 사례2

- AWS에서 서버 상태를 체크하는 Health Check 서비스도 싱글톤으로 구현할 수 있다.
- 예를 들어 LB의 싱글톤 클래스가 서버의 목록을 만들고 목록에서 제거된 서버의 상태는 확인하지 않는 간단한 코드를 구현해 보았다.

```python
class HealthCheck:
    _instances = None

    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, "instance"):
            cls.instance = super().__new__(cls)
        return cls.instance

    def __init__(self):
        self._servers = []

    def add_server(self):
        self._servers.append("Server 1")
        self._servers.append("Server 2")
        self._servers.append("Server 3")
        self._servers.append("Server 4")

    def change_server(self):
        self._servers.pop()
        self._servers.append("Server 5")

hc1 = HealthCheck()
hc2 = HealthCheck()

hc1.add_server()
print("Schedule health check for servers (1)..")

for i in range(4):
    print(f"Checking {hc1._servers[i]}")

hc2.change_server()
print("Schedule health check for servers (2)..")

for i in range(4):
    print(f"Checking {hc2._servers[i]}")

---------------
result
---------------
Schedule health check for servers (1)..
Checking Server 1
Checking Server 2
Checking Server 3
Checking Server 4
Schedule health check for servers (2)..
Checking Server 1
Checking Server 2
Checking Server 3
Checking Server 5
```

- `change_server()` 에 의해 제거 된 서버는 두 번째 서비스가 실행 될 때는 바뀐 목록을 참조한다.
- 상태값을 공유하는 모노스테이트 싱글톤 패턴을 사용해도 되지만, Health Check가 빈번할 경우 리소스 소모가 클 것으로 예상되어 적절하지 않아 보인다.

## 싱글톤 패턴의 장점

- 어플리케이션을 개발할 때 스레드 풀과 캐시, 대화 상자, 레지스트리 설정 등 한 개의 객체만 필요한 경우가 많은데 이러한 경우에 여러 개의 객체를 생성하는 것은 리소스 낭비이고, 싱글톤 패턴이 유리하다.
- 싱글톤은 글로벌 액세스 지점을 제공하는, 단점이 거의 없는 검증된 패턴이다.

## 싱글톤 패턴의 단점

- 전역 변수의 값이 변경되었을 때 사이드 이펙트가 매우 클 수 있다.
- 하나의 객체만을 생성하기 때문에 같은 객체를 참조하는 여러개의 참조자가 생긴다.
    - a = A()
    - b = A()
    - c = A()