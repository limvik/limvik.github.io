---
title: ubuntu 에서 nodejs 최신 버전 설치 및 삭제 방법
layout: post
categories:
- Javascript
- Node.js
tags:
- Javascript
- Node.js
- Ubuntu
date: 2023-03-23 21:05 +0900
---

## Intro
[Next.js 튜토리얼](https://nextjs.org/learn/foundations/from-react-to-nextjs/getting-started-with-nextjs) 진행 중에 Node.js를 설치하라고 하는데, [검색 결과](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)를 따라 그냥 sudo apt install nodejs 를 했더니 Node.js 12 버전이 설치가 됐습니다.
그래서 chat GPT에 Node.js 최신 버전 설치 방법을 물어봤고, 이상 없이 수행됨을 확인하였습니다. 제가 사용하는 Ubuntu 버전은 아래와 같습니다.

```
Ubuntu 22.04.2 LTS (GNU/Linux 5.15.90.1-microsoft-standard-WSL2 x86_64)
```

저는 처음에 sudo apt install nodejs 명령어로 설치 한 후에 그대로 설치 절차를 거치니 정상적으로 동작하지 않아 제거 후에 다시 설치하니 정상적으로 동작하였습니다. 이미 sudo apt install nodejs 명령어를 이용하여 설치하신 분은 삭제 후에 설치 절차를 진행해 보시길 권장드립니다.

되도록 chatGPT에서 얻은 원문 그대로 붙여넣었습니다.

## 설치 절차

To install to the latest version of Node.js in Ubuntu, you can use the NodeSource package repository. Follow these steps:
1. Install the required packages for adding the repository:
```bash
sudo apt-get install curl gnupg2 ca-certificates lsb-release
```
2. Add the NodeSource repository for the latest LTS version (recommended for most users) by running the following command:
```bash
curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
```
For the latest stable version, replace `setup_lts.x` with `setup_16.x`.
(chatGPT가 아직 최신 데이터는 없어서 16버전을 언급하나봅니다. 2023년 3월 23일 기준 [Node.js 공식 홈페이지](https://nodejs.org/en)의 최신 LTS 버전은 18.15.0 입니다.)

3. Update the package list and upgrade the installed packages:
```bash
sudo apt-get update
sudo apt-get upgrade
```
4. Install the latest version of Node.js:
```bash
sudo apt-get install -y nodejs
```
5. Verify the installed version of Node.js:
```bash
node -v
```
This should output the version number of the installed Node.js.

## 삭제 절차
To remove Node.js from Ubuntu, you can use the following steps:

1. First, remove the installed Node.js packages by running the following command:
```bash
sudo apt-get  remove --purge nodejs
```
This will remove the Node.js package along with its configuration files.

2. Next, remove any additional packages that may have been installed alongside Node.js by running the following command:
```bash
sudo apt-get autoremove
```
This will remove any packages that were installed as dependencies of Node.js but are no longer needed.

3. Finally, remove the Node.js repository from your system by deleting the relevant file from the `/etc/apt/sources.list.d/` directory. Use the following command to delete the file:
```bash
sudo rm /etc/apt/sources.list.d/nodesource.list
```
This file contains the NodeSource repository information that was added during installation.

After these steps, Node.js will be completely removed from your Ubuntu system. You can verify this by running the `node -v` command, which should return a command not found error.

## 추가 자료
### npm 최신 버전 설치 절차
To install the latest version of npm (Node Package Manager) in Ubuntu, you can follow these steps:
1.  First, update the package list on your system by running the following command:
```bash
sudo apt-get  update
```
2.  Next, install npm by running the following command:
```bash
sudo apt-get install npm
```
This will install the version of npm that is available in the default Ubuntu repository. However, it might not be the latest version.

3.  To install the latest version of npm, you can use the following command to update npm to the latest version:
```bash
sudo npm install -g npm@latest
```
This will install the latest version of npm globally on your system.

4.  Verify the installed version of npm by running the following command:
```bash
npm -v
```
This should output the version number of the installed npm.

That's it! You now have the latest version of npm installed on your Ubuntu system.
### Node.js 설치 시 패키지 설명
```bash
sudo apt-get install curl gnupg2 ca-certificates lsb-release
```
These packages are required to add the NodeSource repository, which is a third-party repository that provides the latest versions of Node.js for Ubuntu. Here's what each package does:

-   `curl`: A command-line tool for transferring data from or to a server, which is used to download the NodeSource repository configuration script.
    
-   `gnupg2`: A command-line tool for secure communication and data storage, which is used to securely authenticate the NodeSource repository configuration script.
    
-   `ca-certificates`: A package that contains the trusted Certificate Authorities (CA) certificates for validating SSL/TLS connections, which is required to securely download and authenticate the NodeSource repository configuration script.
    
-   `lsb-release`: A package that provides information about the Linux distribution and version, which is used by the NodeSource repository configuration script to determine the correct repository configuration for your Ubuntu version.
    

By installing these packages before adding the NodeSource repository, you ensure that the repository is securely authenticated and that your system can connect to it using SSL/TLS encryption.
