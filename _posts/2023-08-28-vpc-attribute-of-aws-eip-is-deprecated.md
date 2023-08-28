---
layout: post
title: vpc attribute of aws_eip is deprecated
categories:
- Cloud
- Terraform
tags:
- AWS
- Terraform
date: 2023-08-28 21:50 +0900
---
## Intro

클라우드에 배포까지 다 하고 싶어서 DevOps 기초 강의를 듣는 중입니다. 강의 순서가 Terraform 을 먼저 배우게 되어있어서 듣고 있는데, 만든지 시간이 꽤 지난 강의다 보니 Deprecated 된게 있었습니다. AWS SDK가 변경돼서 그런거겠지만, Terraform 사용하지 않으면 마주칠 일 없는 오류인 것 같아서 Terraform 으로 분류합니다.

## 오류 메시지

NAT 게이트웨이에 사용할 aws_eip 설정하는데, `vpc` 속성을 `true`로 설정하니 `Argument is deprecated` 메시지가 시현되었습니다.

```bash
│
│ Warning: Argument is deprecated
│
│   with aws_eip.nat,
│   on vpc.tf line 41, in resource "aws_eip" "nat":
│   41:   vpc = true
│
│ use domain attribute instead
│
│ (and one more similar warning elsewhere)
│
```

vpc = true 에서 단순하게 domain = true 로 바꿨더니 안됩니다.

## 해결책

```bash
domain = "vpc"
```

vpc = true 를 domain = "vpc"로 변경하면 됩니다.

무슨 프로그램인지는 모르겠지만, 감사하게도 본인들의 github issue([링크](https://github.com/aws-ia/terraform-aws-vpc/issues/125))에 해결방법을 알려주신 분이 있었습니다.
