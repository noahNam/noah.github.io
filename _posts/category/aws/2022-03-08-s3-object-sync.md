---
layout: post
title: AWS 계정간 S3 Bucket Sync
description: >
    AWS 계정간 S3 Bucket Sync
sitemap: false
hide_last_modified: false
categories:
  - category
  - aws
---


# AWS 계정간 S3 Bucket Sync

* toc
{:toc .large-only}

## 목적
![](/assets/img/posts/s3_sync/sync1.png){: width="800" height="800"}

- 계정 A의 Bucket(**Source-Bucket**)에 있는 object를 계정 B의 Bucket(**Target-Bucket**)과 Sync를 맞춘다. 즉 object를 복사한다.

## How?

Target-Bucket S3에서 S3 Bucket 권한을 갖은 IAM 객체를 만들고 복사할 객체를 가지고 있는 Source-Bucket의 Bucket Policy를 수정한 후 IAM 객체를 이용하여 AWS CLI를 통해 Object를 복사한다.

1.계정 B의 IAM에서 정책(Policy)를 생성한다. `Source-Bucket`에는 A계정의 Bucket 이름, `Target-Bucket`에는 B계정의 Bucket 이름을 입력한다. 정책 이름은 `s3-sync-policy`로 하였다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::Source-Bucket",
        "arn:aws:s3:::Source-Bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": [
        "arn:aws:s3:::Target-Bucket",
        "arn:aws:s3:::Target-Bucket/*"
      ]
    }
  ]
}
```

2.`s3-sync-policy` 정책을 사용자(User)의 권한에 추가한다. 추가하는 이유는 CLI를 이용하여 Target-Bucket이 있는 S3에 접근을 해야하기 때문이다. 기존 사용하던 사용자(User)가 있을 시 기존 사용자에게 추가, 사용 하던 사용자(User)가 없다면 새로 생성하면 된다.

![](/assets/img/posts/s3_sync/sync2.png){: width="600" height="600"}

3.계정 A의 S3 > Source-Bucket > 권한 > 버킷 정책 으로 접근 하여 아래와 같은 버킷 정책을 등록한다.

![](/assets/img/posts/s3_sync/sync3.png){: width="600" height="600"}

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DelegateS3Access",
      "Effect": "Allow",
      "Principal": {
        "AWS": "Account B's AccountID"
      },
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::Source-Bucket/*",
        "arn:aws:s3:::Source-Bucket"
      ]
    }
  ]
}
```

- `Account B's AccountID` 에는 B계정의 계정ID를 입력한다.
- `Source-Bucket` 에는 A계정의 Bucket 이름을 입력한다.

4.`aws configure` 명령어를 통해 B계정의 IAM 키 값을 등록한다. (복사 주최는 B계정)

5.아래 명령어로 Sync를 진행한다.
    
```bash
$ aws s3 sync s3://Bucket-Source s3://Bucket-Target

copy: s3://Bucket-Source/rest_framework/js/jquery-3.5.1.min.js to s3://Bucket-Target/rest_framework/js/jquery-3.5.1.min.js
copy: s3://Bucket-Source/rest_framework/js/coreapi-0.1.1.js to s3://Bucket-Target/rest_framework/js/coreapi-0.1.1.js

# 디렉토리 Sync
$ aws s3 sync s3://Bucket-Source/디렉토리명 s3://Bucket-Target/디렉토리명
```