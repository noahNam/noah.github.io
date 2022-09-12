---
layout: post
title: "[AWS] Shield"
description: >
    [AWS] Shield
sitemap: false
hide_last_modified: false
categories:
  - category
  - aws
---

# [AWS] Shield

* toc
{:toc .large-only}

## AWS Shield란?

- 관리형 디도스 방어 서비스

## Shield 필요성

- AWS 클라우드 서비스를 채택하는 기업이 폭발적으로 늘어남에 따라 악의적인 공격자와 해커는 다중 네트워크 레이어(Layer 3, Layer 4, Layer 7) 및 대량 DDoS 등 새로운 방식의 공격을 통해 기업의 IT서비스를 위협하고 있다. 이에 대응하기 위해 AWS는 네트워크 경계 수준에서 악성 트래픽을 차단하고 애플리케이션을 보호하기 위해 지속적인 투자를 하고 있으며, 애플리케이션 및 DDoS 공격을 완화하기 위해 AWS Web Application Firewall (AWS WAF) 과 AWS Shield 두 가지 강력한 보안 서비스를 제공하고 있다.

## AWS Shield 두 계층

### Standard Protection

- 추가 비용없이 모든 AWS 고객에게 적용되는 자동 보호 서비스
- 독자적인 BlackWatch 시스템을 이용하여 방어
- 추가적인 방어 여력
- 탐지 및 방어 수준의 지속적인 향상
- AWS cloudfront, Route53서비스에 한하여 L3, L4레벨(네트워크 및 전송 계층)의 DDoss 공격을 보호해 준다.
- AWS 서비스에 적용 : CloudFront 및 Route53

### Advanced Protection

- 추가적인 보호와 기능 및 디점을 제공하는 비용 기반 서비스
- 정교한 대규모 DDos 공격에 대한 추가 보호 및 완화
- WAF와의 통합
- DDos 관련 트래픽 급증시 요금 보호를 제공

## AWS DDos Shield 선택 방법(Standard / Advanced)

### Standard Protection

- 대부분의 일상적인 디도스 공격의 방어를 위해
- AWS 상의 디도스 방어를 강화하기 위한 도구 및 모범사례들을 이용하고자 할 때

### Advanced Protection

- 좀 더 규모가 크고 복잡한 형태의 공격에 대한 추가적인 보호를 위해
- 공격에 대한 가시성 확보를 위해
- 공격으로 인한 손해를 회피하기 위해
- 24X7 기반으로 디도스 전문사에게 복잡한 케이스들을 요청하기 위해