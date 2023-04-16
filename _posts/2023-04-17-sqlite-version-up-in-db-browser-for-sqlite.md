---
layout: post
title: DB Browser for SQLite에서 SQLite version 업그레이드 하는 방법
categories:
- Database
- SQLite
tags:
- Database
- SQLite
date: 2023-04-17 00:18 +0900
---
## Intro
DB Browser for SQLite에 SQLite도 같이 있는건줄 모르고, DB Browser for SQLite 하고 SQLite를 따로 받았습니다. 그래서 SQLite 따로 받은 김에 DB Browser for SQLite에 포함된 SQLite를 버전 업 시키는 방법을 공유합니다.

## 다운로드

제가 사용하는 Windows 11 기준입니다. Windows 다른 버전도 다 될 것으로 생각됩니다.

1. SQLite DLL 다운로드 받기([링크](https://www.sqlite.org/download.html))
![sqlite download page](/assets/img/2023-04-17-sqlite-version-up-in-db-browser-for-sqlite/2023-04-17-sqlite-download-page.png)
2. DB Browser for SQLite 다운로드 받기([링크](https://sqlitebrowser.org/dl/))
	저는 installer 설치를 별로 안좋아해서 zip 파일로 받았습니다.
3. 설치 또는 압축 풀기

## 파일 덮어쓰기

SQLite 홈페이지에서 받은 SQLite DLL 파일을 받아서 압축을 풀었으면, `sqlite3.dll` 파일이 보이실 겁니다.

![sqlite3.dll](/assets/img/2023-04-17-sqlite-version-up-in-db-browser-for-sqlite/2023-04-17-sqlite-dll.png)

이 파일을 DB Browser for SQLite 폴더 안에 그대로 덮어쓰면 됩니다.
![덮어쓰기 경고창](/assets/img/2023-04-17-sqlite-version-up-in-db-browser-for-sqlite/2023-04-17-overwrite.png)

## 버전 확인하기

DB Browser for SQLite 에서 `도움말` 메뉴의 `정보` 를 확인하면, 버전이 옛날 버전임을 확인할 수 있습니다.

![DB Browser for SQLite 정보창](/assets/img/2023-04-17-sqlite-version-up-in-db-browser-for-sqlite/2023-04-17-db-browser-info.png)

SQL 실행 탭으로 이동해서, 아래와 같은 SQL을 입력하면 다운로드 받았던 DLL 버전이 출력된 것을 확인할 수 있습니다.

```sql
select sqlite_version();
```

![DB Browser for SQLite SQL 실행 탭](/assets/img/2023-04-17-sqlite-version-up-in-db-browser-for-sqlite/2023-04-17-select-sqlite-version.png)

## Outro

그런데 어떤 영향이 있을지는 저도 알 수가 없으니 SQLite로 중요한 작업을 하시는 분은 권장드리지 않습니다.
