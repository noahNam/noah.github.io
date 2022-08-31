---
layout: post
title: 도메인 영역 주요 구성요소
description: >
    도메인 영역 주요 구성요소
sitemap: false
hide_last_modified: false
categories:
  - category
  - DDD
---


# 도메인 영역 주요 구성요소

* toc
{:toc .large-only}

## Entity

- `고유의 식별자`를 갖는 객체로 `자신의 라이프 사이클`을 갖는다.
- Order, User, Product와 같이 `도메인의 고유한 개념을 표현`한다.
- 도메인 모델의 데이터를 포함하며 해당 데이터와 관련된 기능을 함께 제공한다.

## Value

- `고유의 식별자를 갖지 않는 객체`
- 주로 개념적으로 `하나인 값을 표현할 때 사용`한다. 배송지 주소 표현을 위한 주소, 주문을 위한 구매금액이 해당한다.
- 엔티티의 속성으로 사용되어지고 다른 밸류 타입의 속성으로도 사용할 수 있다.
- <strong>`불변으로 구현할 것을 권장하며, 이는 엔티티의 밸류 타입의 데이터를 변경할 때는 객체 자체를 완전히 교체 해야하는 것을 의미한다.`</strong>

## Aggregate

- 연관된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것이다.
- 예를 들어 주문과 관련된 Order Entity, OrderLine Value, Orderer Value 객체를 '주문' Aggeregate로 묶을 수 있다.


## Repository

- `도메인 모델의 영속성`을 처리한다.
- DBMS 테이블에서 엔티티 객체를 로딩하거나 저장하는 기능을 제공한다.

## Domain Service

- 특정 엔티티에 속하지 않은 도메인 로직을 제공한다.
- '할인 금액 계산'은 상품, 쿠폰, 회원, 구매금액 등 다양한 조건을 이용해서 구현하게 되는데, 이렇게 도메인 로직이 여러 엔티티와 밸류를 필요로 하면 도메인 서비스에서 로직을 구현한다.

## 구성요소 별 개요
### Entity와 Value

- DB테이블의 엔티티와 도메인 모델의 엔티티는 다르다.
  - 두 모델의 가장 큰 차이점은 도메인 모델의 엔티티는 데이터와 함께 도메인 기능을 함께 제공한다는 점이다.
  - 예를 들어 주문을 표현하는 엔티티는 주문과 관련된 데이터뿐만 아니라 배송지 주소 변경을 위한 기능을 함께 제공한다.


```python
class OrderEntity(BaseModel):
    # 주문 도메인 모델의 데이터
    order_no: int
    orderer: OrdererValue
    order_line: List[OrderLineValue]
    shipping_info: ShippingInfoValue
    ...

    # 도메인 모델 엔티티는 도메인 기능도 함께 제공
    def change_shipping_info(self, new_shipping_info: ShippingInfoValue):
        self._set_shipping_info(new_shipping_info)

    def _set_shipping_info(self, new_shipping_info: ShippingInfoValue):
        ...
    
    def order(self, order_line, shipping_info):
        self._set_order_lines(order_line)
        ...

class ShippingInfoValue(BaseModel):
    # receiver에 대한 하나의 개념을 위한 value
    receiver_name: str
    receiver_phone_number: str
    shipping_address_1: str
    shipping_address_2: str
    shipping_zip_code: str

class OrdererValue(BaseModel):
    # 주문자에 대한 하나의 개념을 위한 value
    name: str
    email: str

```

  - 도메인 모델의 엔티티는 `단순히 데이터를 담고 있는 데이터 구조라기보다는 데이터와 함께 기능을 제공하는 객체`이다.
  - <u>도메인 관점에서 기능을 구현하고 기능 구현을 캡슐화해서 데이터가 임의로 변경되는 것을 막는다.</u>
  - 또 다른 차이점은 `도메인 모델의 엔티티는 두 개 이상의 데이터가 개념적으로 하나인 경우 밸류 타입을 이용해서 표현`할 수 있다는 것이다.
    - 위 코드에서 주문자를 표현하는 `orderer`는 밸류 타입으로 주문자 이름과 이메일 데이터를 포함할 수 있다.
    - 위에서 value에 대해 정의할 때 이야기 했던 원칙을 지키는 것을 권장한다. (<strong>`불변으로 구현할 것을 권장하며, 이는 엔티티의 밸류 타입의 데이터를 변경할 때는 객체 자체를 완전히 교체 해야하는 것을 의미한다.`</strong>)

        ```python
        # 예를 들어 배송지 정보를 변경하는 코드는 기존 객체의 값을 변경하지 않고 다음과 같이 새로운 객체를 필드에 할당한다.
        
        # GOOD
        class OrderEntity(BaseModel):
            # 주문 도메인 모델의 데이터
            shipping_info: ShippingInfoValue
            ...

            # 도메인 모델 엔티티는 도메인 기능도 함께 제공
            def change_shipping_info(self, new_shipping_info: ShippingInfoValue):
                self._set_shipping_info(new_shipping_info)

            def _set_shipping_info(self, new_shipping_info: ShippingInfoValue):
                if not new_shipping_info
                    raise IllegalArgumentException()
                # 밸류 타입의 데이터를 변경할 때는 새로운 객체로 교체한다.
                self.shipping_info = new_shipping_info

        # BAD
        class OrderEntity(BaseModel):
            # 주문 도메인 모델의 데이터
            shipping_info: ShippingInfoValue
            ...

            # 도메인 모델 엔티티는 도메인 기능도 함께 제공
            def change_shipping_info(self, shipping_info: ShippingInfoValue):
                self.shipping_info.receiver_name = '홍길동'
                ...
        
        ```
  
### 애그리거트란?
- 애그리거트는 관련 객체를 하나로 묶은 군집이다.
- 대표적인 예가 주문인데, 주문이라는 도메인 개념은 '주문', '배송지 정보', '주문자', '주문 목록', '총 결제 금액'의 하위 모델로 구성된다.
- 이 하위 개념을 표현한 모델을 하나로 묶어서 '주문'이라는 상위 개념으로 표현할 수 있다.
- 애그리거트를 사용하면 개별 객체가 아닌 애그리거트 간의 관계로 도메인 모델을 이해하고 구현하게 되며, 이를 통해 큰 틀에서 도메인 모델을 관리할 수 있다.

### Aggregate가 사용되어지는 이유
- 도메인이 커질수록 개발할 도메인 모델도 커지면서 많은 엔티티와 밸류가 출현한다.
- 엔티티와 밸류 개수가 많아질수록 모델은 점점 더 복잡해진다.
  - 이런 경우 개발자가 전체 구조가 아닌 한 개 엔티티와 밸류에만 집중하는 상황이 발생한다.
  - 이 때 상위 수준에서 모델을 관리하지 않고 개별 요소에만 초점을 맞추다 보면, 큰 수준에서 모델을 이해하지 못해 큰 틀의 모델을 관리할 수 없는 딜레마에 빠질 수 있다.
- `도메인 모델에서 전체 구조를 이해하는 데 도움이 되는 것이 애그리거트이다.`


### Aggregate 특성
- 애그리거트는 군집에 속한 객체를 관리하는 루트 엔티티를 갖는다.
- 루트 엔티티는 애그리거트에 속해 있는 엔티티와 밸류 객체를 이용해서 애그리거티거트가 구현해야 할 기능을 제공한다.
- 애그리거르를 사용하는 코드는 애그리거트 루트가 제공하는 기능을 실행하고 애그리거트 루트를 통해서 간접적으로  애그리거트 내의 다른 엔티티나 밸류 객체에 접근한다. 이것은 애그리거트의 내부 구현을 숨겨서 애그리거트 단위로 구현을 캡슐화할 수 있도록 돕는다.

- 위 UML에서 애그리거트 루트인 Order는 주문 도메인 로직에 맞게 애그리거트의 상태를 관리한다. 예를 들어 Order의 배송지 정보 변경 기능은 배송지를 변경할 수 있는지 확인한 뒤에 배송지 정보를 변경한다.
    ```python
    class OrderRoot(BaseModel):
        order_no: str
        total_amounts: Money
        orderer: OrdererValue
        order_line: List[OrderLineValue]
        shipping_info: ShippingInfoValue
        
        def change_shipping_info(self, new_shipping_info: ShippingInfoValue):
            self._check_shipping_info_chanagealbe()  # 배송지 변경 가능 여부 확인 
            self.shipping_info = new_shipping_info

        def _check_shipping_info_chanagealbe(self):
            # 배송지 정보를 변경할 수 있는지 여부를 확인하는 도메인 규칙 구현

- _check_shipping_info_chanagealbe()는 도메인 규칙에 따라 배송지를 변경할 수 있는지 확인하고 이미 배송이 시작된 경우 익셉션을 발생하는 식으로 도메인 규칙을 구현한다.
- `주문 애그리거트는 Order를 통하지 않고 shipping_info를 변경할 수 있는 방법을 제공하지 않는다.` 즉, 배송지를 변경하려면 루트 엔티티인 Order를 사용해야 하므로 배송지 정보를 변경할 때에는 Order가 구현한 도메인 로직을 항상 따르게 된다.

### Aggregate 구현 시 주의점
- 애그리거트에 구현을 어떤식으로 하는지에 따라 복잡도가 증가한다.
- 트랜젝션 범위가 달라지기도 한다.
- 선택한 구현 기술에 따라 애그리거트 구현에 제약이 생기기도 한다.

### 레포지토리
- 도메인 객체의 지속적 사용을 위해 RDBMS와 같은 물리적인 저장소에 도메인 객체를 보관해야 하는데, 이를 위한 도메인 모델이 레포지터리이다.
- 엔티티나 밸류가 요구사항에서 도출되는도메인 모델이라면 레포지터리는 구현을 위한 도메인 모델이다.
- `레포지터리는 애그리거트 단위로 도메인 객체를 저장하고 조회하는 기능을 정의한다.` 
- 주문 애그리거트를 위한 레포지터를 다음과 같이 정의할 수  있다.
    ```
    python
    class OrderRepository:
        @abstractmethod
        def find_by_number(order_number: int)
    
        @abstractmethod
        def save(order: Order)

        @abstractmethod
        def delete(order: Order)
    ```
    - OrderRepository