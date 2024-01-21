---
layout: post
title: API Gateway
categories:
- Cloud
- Cloud Native
tags:
- Cloud Native
- API Gateway
date: 2024-01-21 21:12 +0900
---
## Intro

Cloud Native 앱을 만들려고 하다보니, API Gateway가 빠질 수가 없습니다. 이름이 직관적이라 대충 느낌적인 느낌은 옵니다. 그래서 느낌이 맞는지 확인하고 조금 더 구체적으로 찾아보기 위해, API Gateway가 무엇이고 어떤 종류가 있는지 간단하게 살펴봅니다.

## API Gateway 의 정의

API Gateway를 검색해보면, 클라우드 서비스(AWS, Azure, GCP 등) 마다 API Gateway 서비스를 제공하고 있어서 손쉽게 설명을 찾아볼 수 있습니다.

그런데 각 서비스에 치우쳐서 설명되어 있어, CNCF(Cloud Native Computing Foundation) 용어집에 적혀 있는 일반적인 정의([링크](https://glossary.cncf.io/api-gateway/))를 살펴보겠습니다.

> An API gateway is a tool that aggregates unique application APIs, making them all available in one place. It allows organizations to move key functions, such as authentication and authorization or limiting the number of requests between applications, to a centrally managed location. An API gateway functions as a common interface to (often external) API consumers.
>
> API 게이트웨이는 고유한 애플리케이션 API를 통합하여 한 곳에서 모두 사용할 수 있도록 하는 도구입니다. 이를 통해 조직은 인증 및 권한 부여 또는 애플리케이션 간 요청 수 제한과 같은 주요 기능을 중앙에서 관리하는 위치로 옮길 수 있습니다. API 게이트웨이는 (주로 외부) API 소비자에 대한 공통 인터페이스 역할을 합니다.

## API Gateway 의 위치

다이어그램으로 보자면, 일반적으로 아래와 같이 API 앞단에 위치하고 있습니다.

![01.Cloud Native Design](/assets/img/2024-01-21-api-gateway/01.cloud-native-design.png)

출처: [Microsoft - Introduction to cloud-native applications](https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/introduction)

처음에는 'API Gateway가 굳이 필요한가?' 잠시 생각했지만, 인증에 한정해서만 생각해봐도 각 서비스마다 인증 로직을 별도로 만들게 아니면 중앙화된 인증 시스템이 필요합니다. 그리고 추측해 보자면, 앞서 정의에서 언급된 요청 수 제한과 같이 중앙화하면 편리한 것들을 모으다보니 그게 API Gateway가 된것이 아닐까? 하는 생각도 듭니다.

## API Gateway 도구 및 서비스

대문 역할을 하는 API Gateway 가 중요한 역할을 하는 만큼 다양한 도구와 서비스가 존재합니다. 대표적으로 포트폴리오용 프로젝트에서 많이 보이는 [NGINX](https://www.nginx.com/learn/api-gateway)가 생각납니다. 간단하게 어떤 도구와 서비스들이 있는지 살펴보겠습니다.

### Cloud Native Landscape 목록

먼저 CNCF 에서 관리하는 목록에 등재된 API Gateway 도구 및 서비스 종류를 살펴보겠습니다.

![02.API Gateway lists in CNCF Landscape](/assets/img/2024-01-21-api-gateway/02.api-gateway-in-cloud-native-landscpae.png)

출처: [CNCF Landscape](https://cncf.landscape2.io/)

이 목록에 있는 것만 무려 22개 입니다. Azure는 있는데 AWS는 없다니, 돈을 안냈나...?

### Spring

제가 주로 사용하는 Spring 프로젝트에도 [Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/index.html) 라는 API Gateway를 구현하는 프로젝트가 존재합니다.

당연히 요청을 많이 처리해야되니 Reactive 버전만 존재할 줄 알았는데, Servlet 버전도 존재합니다. [Release Note](https://github.com/spring-cloud/spring-cloud-gateway/releases?q=mvc&expanded=true)를 찾아보니 Servlet 버전은 작년 8월 4.1.0에서 추가되었습니다.

![03.Spring Cloud Gateway Release Note](/assets/img/2024-01-21-api-gateway/03.spring-cloud-gateway-mvc.png)

Spring Cloud Gateway를 사용하려면 어떤 버전이 더 나은지 비교하는데 시간이 또 걸리겠습니다.

API Gateway 를 직접 구현할 때 고려해야할 사항을 간단하게 생각해 보겠습니다. 요청을 혼자서 처리하는데 장애가 발생하면, 단일 장애점(SPoF; Single Point of Failure)이 될 수 있기 때문에 최소 2개는 운영해야 합니다. 2개 이상 운영한다면, Active-Stanby 방식으로 장애 시 전환을 하던가 Active-Active 방식으로 로드 밸런서를 이용해 부하를 분산하는게 필요해 보입니다.

학습용이라면 도전적이고 좋은데, 실 운영을 혼자서 한다면 관리가 부담스럽습니다. 이럴 때는 클라우드 업체에서 제공하는 API Gateway 서비스를 사용하는 것이 더 적합해 보입니다. 

### 클라우드 API Gateway 서비스

최근 개인적으로 학습 중인 [AWS API Gateway](https://aws.amazon.com/ko/api-gateway/)를 살펴보겠습니다.

#### AWS API Gateway

AWS API Gateway는 프리티어로 제공되는 용량이 상당하다보니 포폴용이나 개인 프로젝트에 사용하기에도 좋아보입니다.

> API Gateway 프리 티어에는 최대 12개월 동안 매달 HTTP API 호출 100만 개, REST API 호출 100만 개, 메시지 100만 개, 연결 시간 750,000분이 포함됩니다.

최근 [Coursera 강의](https://www.coursera.org/learn/developing-applications-in-python-on-aws)에서 SAM(Serverless Application Model) 서비스를 기반으로 API Gateway와 AWS Lambda를 연동하는 실습을 해봤는데, 아래와 같이 YAML 설정으로 구성이 가능합니다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  add-service-sam
  Sample SAM Template for add-service-sam

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
  Function:
    Timeout: 3

Resources:
  AddServiceFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: add/
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
        - x86_64
      Events:
        Sum:
          Type: HttpApi
          Properties:
            Path: /sum
            Method: get
            ApiId: !Ref HttpApi
          
  HttpApi:
    Type: AWS::Serverless::HttpApi
    Properties:
      StageName: "$default"



Outputs:
  AddServiceFunction:
    Description: "Add Service Lambda Function ARN"
    Value: !GetAtt AddServiceFunction.Arn
  AddServiceFunctionIamRole:
    Description: "Implicit IAM Role created for Add Service function"
    Value: !GetAtt AddServiceFunction.Arn
  HttpApiUrl:
    Description: URL of your API endpoint
    Value:
      Fn::Sub: 'https://${HttpApi}.execute-api.${AWS::Region}.${AWS::URLSuffix}/'
```

실습이 아쉽게도 `이거 붙여넣으면 동작됨 ㅇㅇ` 스타일의 실습이라 잘 활용하려면 학습이 필요합니다.

여튼, 프리티어 요건에서 보았듯 건당 과금이므로 아직 요청 수가 많지 않은 서비스에서 사용하기에 적합해 보입니다. 

반면에 [Azure API Management](https://azure.microsoft.com/en-us/pricing/details/api-management/)는 시간당 과금이라 서비스가 커지고 요청수가 크게 증가한다면, 비용을 계산해보고 저렴한 쪽으로 옮겨타는 것도 고려해볼 수 있겠습니다. 그런데 클라우드는 인지하기 어려운 숨겨진 비용도 많아서 옮긴다는게 쉽지가 않은 것 같기는 합니다.

## Outro

정말 간단하게 API Gateway 가 무엇이고, 어떤 종류가 있는지 살펴보았습니다. 취업용 필기 시험 공부 중이라 깊게 살펴볼 시간이 부족해 아쉽습니다. 시험 끝나고 프로젝트 하면서 Spring Cloud Gateway를 사용하게 되면 관련 내용 공유해 보겠습니다.

글을 쓰고나서 토스 기술 블로그에서 API Gateway 관련 잘 정리된 글([링크](https://toss.tech/article/slash23-server))을 찾았습니다. 토스에서의 API Gateway 사용 사례를 소개하고있어 궁금하신 분들은 참고해보시면 좋을 것 같습니다.

## 참고자료

- [API Gateway - Cloud Native Glossary](https://glossary.cncf.io/api-gateway/)
- [Introduction to cloud-native applications - Microsoft](https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/introduction)
