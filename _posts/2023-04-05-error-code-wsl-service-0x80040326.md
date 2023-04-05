---
layout: post
title: 'Error code: Wsl/Service/0x80040326'
categories:
- WSL
tags:
- WSL
- Error
date: 2023-04-05 22:35 +0900
---
## 요약
```powershell
wsl --update
```
위 명령어를 입력하여 wsl을 업데이트 합니다.

## Intro
WSL(Windows Subsystem for Linux) 을 주로 사용하는데, 일주일 전 즈음 갑자기 오류가 발생해서 당황했었습니다. Error code를 검색하니 역시 질문과 답이 올라와 있어서 손쉽게 해결할 수 있었습니다.

## 해결
파워쉘(PowerShell) 이나 명령 프롬프트(Command Prompt)에서 아래 명령어를 입력하여 WSL을 업데이트하면 완료됩니다.
```powershell
wsl --update
```
저는 명령어를 입력했을 때 업데이트 완료 메시지가 나왔었는데도 실행이 안돼서, 다시 업데이트 했습니다.

그래서 확인을 위해 설치하고나서 다시 한 번 명령어를 입력해보면, 아래와 같은 메시지가 표시되고, 이상 없이 설치한 WSL 배포판이 실행됩니다.

![PowerShell에서 wsl 명령어 실행](/assets/img/2023-04-05-powershell.png)

## 참고 자료
- [https://superuser.com/questions/1776464/ubuntu-crashes-on-windows-0x80040326-wsl-error](https://superuser.com/questions/1776464/ubuntu-crashes-on-windows-0x80040326-wsl-error)
- [https://learn.microsoft.com/en-us/answers/questions/1194533/how-to-resolve-wsl-service-createinstance-0x800403](https://learn.microsoft.com/en-us/answers/questions/1194533/how-to-resolve-wsl-service-createinstance-0x800403)
