---
layout: post
title: Aggregate Root
description: >
    애그리거트 루트
sitemap: false
hide_last_modified: false
categories:
  - category
  - DDD
---


# Aggregate Root

* toc
{:toc .large-only}

## 애그리거트 루트
![Aggregate](/assets/img/posts/ddd/Aggregate.png)
[Aggregate Post 링크](https://noahnam.github.io/category/ddd/2022-01-24-aggregate/)

- 주문 애그리거트는 다음을 포함한다.
  - 총 금액인 `totalAmounts`를 갖고 있는 `Order` 엔티티
  - 개별 구매 상품의 갯수인 `quantity`와 금액인 `price`를 갖고 있는 `OrderLine` 밸류

- 애그리거트는 여러 객체로 구성되기 때문에 한 객체만 상태가 정상이면 안 된다. 즉, 도메인 규칙을 지키려면 애그리거트에 속한 모든 객체가 정상 상태를 가져야 한다.
  - 구매할 상품의 갯수를 변경하면 `OrderLine` 밸류의 `quantity`를 변경하고 `Order`엔티티의 `totalAmounts`도 변경해야 한다.
  - 그렇지 않으면 도메인 규칙을 어기고 데이터 일관성이 깨진다.

- 이처럼 <strong>`애그리거트에 속한 모든 객체가 일관된 상태를 유지하려면 애그리거트 전체를 관리할 주체가 필요한데, 이 책임을 지는 것이 바로 애그리거트의 루트 엔티티이다.`</strong>
  - 주문 애그리거트에서 루트 엔티티는 `Order` 이다. 
  - 애그리거트에 속한 객체는 애그리거트 루트 엔티티에 직접 또는 간접적으로 속하게 된다.
![Aggregate Root Entity](/assets/img/posts/ddd/애그리거트루트.png)

## 도메인 규칙과 일관성
 - 애그리거트 루트 엔티티의 핵심 역할은 애그리거트의 일관성이 깨지지 않도록 하는 것이다.
 - 이를 위해 애그리거트 루트는 애그리거트가 제공해야 할 도메인 기능을 구현한다.
   - 예로, 주문 애그리거트는 배송지 변경, 상품 변경과 같은 기능을 제공하고, 애그리거트 루트인 `Order`가 이 기능을 구현한 메소드를 제공한다.
 - 이렇게 구현한 메소드는 도메인 규칙에 따라 애그리거트에 속한 객체의 일관성이 깨지지 않도록 구현해야 한다.
   - 예로, 배송이 시작되기 전까지만 배송 정보를 변경할 수 있다는 규칙이 있다면, 
   - 애그리거트 루트인 `Order`의 `change_shipping_info`메소드는 이 규칙에 따라 배송 시작 여부를 확인하고 배송지 정보를 변경할 수 있도록 구현해야 한다.
    ```python
    class Order(BaseModel):
        state: bool
        ...

        # 애그리거트 루트는 도메인 규칙을 구현한 기능을 제공한다.
        def change_shipping_info(self, new_shipping_info: ShippingInfoValue):
            self._verify_not_yet_shipped()
            self._set_shipping_inf(new_shipping_info)

        def _verify_not_yet_shipped(self):
            if state != OrderState.PAYMENT_WAITING and state != OrderState.PREPARING
              raise new IllegalStateException("already shipped")
    ```
- 애그리거트 외부에서 애그리거트에 속한 객체를 직접 변경하면 안된다. 이것은 애그리거트 루트가 강제하는 규칙을 적용할 수 없어 모델의 일관성을 깨는 원인이 된다.
    ```python
    # 1번 코드
    si: ShippingInfoValue = order.get_shipping_info()
    si.set_address(new_address)

    # 2번 코드
    si: ShippingInfoValue = order.get_shipping_info()
    if state != OrderState.PAYMENT_WAITING and state != OrderState.PREPARING
      raise new IllegalStateException("already shipped")
    si.set_address(new_address)
    ```
  - 1번 코드는 애그리거트 루트인 `Order`에서 `ShippingInfoValue`를 가져와 직접 정보를 변경하고 있다.
  - 주문 상태에 상관없이 배송지 주소를 변경하는데, 이는 처음에 정한 요구사항 규칙을 무시한채 직접 DB 테이블의 데이터를 수정하는 것과 같은 결과를 만든다.
  - 즉, <strong>`논리적인 데이터 일관성이 깨지게 된다.`</strong>
  - 2번 코드 역시 서비스레이어에서 validation checking을 하지만 동일한 검사로직을 중복으로 할 가능성이 크기 때문에 유지보수에 불리하다.
  
- <strong><u>불필요한 중복을 피하고 애그리거트 루트를 통해서만 도메인 로직을 구현하는 것이 좋은데 이를 위해 2가지를 꼭 지키는 것이 좋다.</u></strong>
  - <strong>`단순히 필드를 변경하는 set 메소드는 공개(public) 범위로 만들지 않는다.`</strong>
    - public 형태의 set 메소드는 도메인의 의미나 의도를 표현하지 못하고, 도메인 로직을 도메인 객체가 다른 레이어로 분산 시킨다.
    - 도메인 로직이 한 곳에 응집되지 않으므로 코드를 유지 보수할 때에도 시간이 오래 걸린다.
    - 또한, set 형식의 이름보다는 cancel이나 change처럼 의미가 더 잘 드러나는 이름을 사용하는 것이 좋다.


  - <strong>`밸류 타입은 불변으로 구현한다.`</strong>
     - 밸류 객체의 값을 변경할 수 없으면 애그리거트 루트에서 밸류 객체를 구해도 애그리거트 외부에서 밸류 객체의 상태를 변경할 수 없다.
     - 밸류 객체가 불변이면 밸류 객체의 값을 변경하는 방법은 새로운 밸류 객체를 할당하는 것 뿐이다.
     - 아래 코드를 보면 애그리거트 루트가 제공하는 `change_shipping_info` 함수에 새로운 밸류ㅜ 객체를 전달해서 값을 변경하는 방법밖에 없다.
    
        ```python
        from pydantic.dataclasses import dataclass

        @dataclass(frozen=True)
        class ShippingInfoValue(BaseModel):
            _address: str
            _message: str | None

        @dataclass(frozen=True)
        class OrderEntity(BaseModel):
          ...
          _shipping_info_value: ShippingInfoValue | None

          def change_shipping_info(self, new_shipping_info):
            object.__setattr__(self, '_shipping_info_value', new_shipping_info)

        # Bad
        class OrderService:
          ...
          si: ShippingInfoValue = order.get_shipping_info()
          si._shipping_info_value = new_address
          # cannot assign to field '_shipping_info_value'

        # Good
        class OrderService:
          ...
          si: ShippingInfoValue = order.get_shipping_info()
          si.change_shipping_info(new_address)
        ```