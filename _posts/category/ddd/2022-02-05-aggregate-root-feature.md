---
layout: post
title: Aggregate Root Feature Implementation
description: >
    애그리거트 루트 기능 구현
sitemap: false
hide_last_modified: false
categories:
  - category
  - DDD
---


# 애그리거트 루트 기능 구현

* toc
{:toc .large-only}

## 애그리거트 루트의 기능 구현
[Aggregate Root Post 링크](https://noahnam.github.io/category/ddd/2022-01-28-aggregate-root/)

- 애그리거트 루트는 애그리거트 내부의 다른 객체를 조합해서 기능을 완성한다.
  - `Order`는 총 주문 금액을 구하기 위해 `OrderLine` 을 사용한다.
    ```python
    class OrderEntity(BaseModel):
        __total_amounts: MoneyValue = PrivateAttr()
        __order_lines: List[OrderLineValue] = PrivateAttr()

        def __init__(self, **data: Any) -> None:
            super().__init__(**data)

        def calculate_total_amounts(self) -> None:
            sum = 0
            for order_line in self.__order_lines:
                sum += order_line.price * order_line.qty

            money_value = MoneyValue()
            money_value.create(amounts=sum)
            self.__total_amounts = money_value
      ```
  - `Member`의 애그리거트 루트는 암호를 변경하기 위해 Password 객체에 암호가 일치하는지를 아래와 같이 확인할 수 있다.
    ```python
    class MemberEntity(BaseModel):
        __password: PasswordValue = PrivateAttr()

        def change_password(self, current_password: str, new_password: str) -> None:
          if current_password != new_password:
            raise PasswordNotMatchException()
          
          self.__password = PasswordValue(password=new_password)
    ```

### 팀 표준이나 구현 기술의 제약으로 `OrderLineValue` 나 `PasswordValue` 를 불변으로 구현할 수 없다면 변경 기능을 패키지나 protected 범위로 한정해서 외부에서 실행할 수 없도록 제한하는 방법도 있다.