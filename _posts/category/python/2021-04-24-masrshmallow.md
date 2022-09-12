---
layout: post
title: masrshmallow
description: >
    masrshmallow
sitemap: false
hide_last_modified: false
categories:
  - category
  - python
---

# Marshmallow

* toc
{:toc .large-only}


## What can you do with the marshmallow?

- Python 객체 → 기본 Python 데이터 타입으로 변환하는 라이브러리
- ORM / ODM 프레임워크에 구애받지 않는다.

1. serializing : Python 객체를 기본 Python 데이터 타입으로 직렬화 한다.
2. deserializing : 입력 데이터를 Python 객체로 역직렬화 한다
3. validation : 입력 데이터의 유효성을 검사한다.

### Serializing Objects ("Dumping")

- 스키마의 메서드에 객체를 전달하여 객체를 직렬화 한다 .

```python
import datetimeas dt

# User 모델 작성
class User:
    def __init__(self, name, email):
				self.name = name
	      self.email = email
        self.created_at = dt.datetime.now()

		def __repr__(self):
			return "<User(name={self.name!r})>".format(self=self)
```

```python
from marshmallow import Schema, fields

# 속성이름을 Field 객체에 맵핑하는 스키마 작성
class UserSchema(Schema):
    name = fields.Str()
    email = fields.Email()
    created_at = fields.DateTime()
```

```python
from pprintimport pprint

user = User(name="Monty", email="monty@python.org")
schema = UserSchema()
result = schema.dump(user)
pprint(result)
# {
#  "name": "Monty",
#  "email": "monty@python.org",
#  "created_at": "2014-08-17T14:54:16.049594+00:00"
# }

# dumps를 이용하면 json인코딩 문자열로 직렬화 할 수도 있다.
json_result = schema.dumps(user)
pprint(json_result)
# '{"name": "Monty", "email": "monty@python.org", "created_at": "2014-08-17T14:54:16.049594+00:00"}'
```

### Deserializing to Objects ("Loading")

- 입력 dict()의 유효성을 검사하고 스키마의  load 메서드에 전달한 후 최종적으로 @post_load가 데코레이팅 되어 역직렬화 한다.

```python
from marshmallow import Schema, fields, post_load

class UserSchema(Schema):
    name = fields.Str()
    email = fields.Email()
    created_at = fields.DateTime()

    @post_load
    def make_user(self, data, **kwargs):
        return User(**data)
```

```python
user_data = {"name": "Ronnie", "email": "ronnie@stones.com"}
schema = UserSchema()
result = schema.load(user_data)
print(result)  # => <User(name='Ronnie')>
```

### Validation

```python
from pprint import pprint
from marshmallow import Schema, fields, ValidationError

class UserSchema(Schema):
    name = fields.String(required=True)
    email = fields.Email()

user_data = [
    {"email": "mick@stones.com", "name": "Mick"},
    {"email": "invalid", "name": "Invalid"},  # invalid email
    {"email": "keith@stones.com", "name": "Keith"},
    {"email": "charlie@stones.com"},  # missing "name"
]

try:
    UserSchema(many=True).load(user_data)
except ValidationError as err:
    pprint(err.messages)
    # {
	#	1: {'email': ['Not a valid email address.']},
    #   3: {'name': ['Missing data for required field.']}
    # }
```

## 참고 자료

[Quickstart - marshmallow 3.11.1 documentation](https://marshmallow.readthedocs.io/en/stable/quickstart.html#serializing-objects-dumping)

[Python 객체에서 데이터 타입으로 바꾸는 marshmallow 라이브러리 알아보기](https://minwook-shin.github.io/python-converting-object-to-datatype-using-marshmallow/)

[Flask-marshmallow로 편하게 데이터~객체 관리하기](https://livlikwav.github.io/flask/Flask-marshmallow/)