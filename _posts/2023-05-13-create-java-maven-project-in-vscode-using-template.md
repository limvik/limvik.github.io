---
layout: post
title: VS Code 에서 Java Maven 프로젝트 생성하기
categories:
- Tools
tags:
- Java
- Maven
- VSCode
- WSL
date: 2023-05-13 11:22 +0900
---
## Intro

VS Code로 Java 프로젝트를 생성하면서 사용했던 방법을 공유합니다. Maven 프로젝트 템플릿 툴인 Archetype을 사용하였으며, Windows 와 WSL(Ubuntu) 환경에서 진행하였습니다. WSL에 open JDK 11과 17, 그리고 maven 3.6.3 버전이 설치된 상태입니다.

## 1. Extension Pack for Java 설치하기

먼저 VS Code 에서 Extensions 메뉴(단축키: ctrl + shift + x)로 이동하여, `Extension Pack for Java`를 설치합니다.

![Extension Pack for Java 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/01.install-vscode-extension-for-java.png)

## 2. Maven 프로젝트 생성하기

### Command Palette 열기

단축키 `Ctrl + shift + p` 를 눌러 Command Palette를 열거나

![VS Code 메인 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/02.blank-vscode.png)

`View` 메뉴를 선택해서 Command Palette를 엽니다.

![VS Code View 메뉴 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/03.command-palette-in-view-menu.png)

### create java project

Command Palette 가 열렸으면, `create java project` 를 찾아서 선택합니다.

![command palette 에서 create java project 입력한 결과 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/04.create-java-project-in-commands.png)

### Create from Archetype

그리고 목적에 맞게 Maven을 선택합니다.

![command palette 에서 Maven 빌드 툴 선택하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/05.maven-for-java-in-command-palette.png)

Maven 선택 후 Command Palette 에서는 archetype의 종류를 선택하는데, 여기서는 maven-archetype-quickstart를 선택하겠습니다. 

![command palette 에서 maven-archetype-quickstart 선택하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/06.maven-archetype-quickstart.png)

> Archetype은 공식 홈페이지([링크](https://maven.apache.org/archetype/index.html))에 따르면, `Maven project templating toolkit` 라고 소개하고 있습니다. 하지만, Maven Repository([링크](https://mvnrepository.com/artifact/org.apache.maven.archetypes/maven-archetype-quickstart))에서 2018년 12월 이후로 업데이트 되지 않고 있습니다. 그래도 공식 홈페이지에서는 Archetype이 Best Practice라고 소개하고 있고, 저 처럼 초보자가 완전히 바닥부터 프로젝트를 구성하는 것 보다는 필요한 것만 조금 수정하면 되니 유용한 것 같습니다.

그리고 버전을 선택해 줍니다. 특별한 이유가 없다면 그나마 최신 버전인 1.4를 선택합니다.

![command palette 에서 archetype 버전 선택하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/07.maven-archetype-quickstart-version.png)

### 프로젝트 정보 입력하기

버전을 선택하고 나면 프로젝트와 관련된 정보를 입력하기 시작합니다.

먼저 group Id는 프로젝트의 도메인을 역순으로 입력합니다. 예를 들어 사진과 같이 com.limvik 으로 입력할 수 있습니다.

![command palette 에서 group Id 입력하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/08.maven-project-group-id.png)

group Id를 입력하고 `Enter` 키를 눌러주면 artifact Id를 입력하는 화면으로 넘어갑니다.

![command palette 에서 artifact Id 입력하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/09.maven-project-artifact-id.png)

참고로, 이번에 사용한 maven archetype quickstart 의 artifact Id는 maven-archetype-quickstart 입니다.

artifact Id에 대한 자세한 설명은 GPT-4의 설명으로 대체합니다.

>`프로젝트_아티팩트ID`는 프로젝트의 고유한 식별자로 사용됩니다. 이것은 프로젝트의 이름을 나타내며, 일반적으로 소문자와 하이픈(-)을 사용하여 표현됩니다. 아티팩트 ID는 프로젝트의 JAR 파일, WAR 파일 등의 이름에 사용되며, Maven 저장소에서 의존성을 찾을 때 사용되는 식별자입니다.
>
>아티팩트 ID를 선택할 때 다음 가이드라인을 따르는 것이 좋습니다:
>
>1. 소문자를 사용하십시오. 대문자는 피하십시오.
>2. 띄어쓰기는 피하고, 대신 단어 사이에 하이픈(-)을 사용하십시오.
>3. 프로젝트의 목적이나 기능을 간결하게 설명하는 이름을 사용하십시오.
>
>예를 들어, 웹 애플리케이션을 개발하는 프로젝트라면 `web-app`, `my-web-app`, `online-store`와 같은 이름을 사용할 수 있습니다. 이렇게 하면 프로젝트의 이름이 의미 있는 동시에 식별이 쉬워집니다.

### 프로젝트 설치 경로 지정 및 버전 설정

artifact Id 까지 입력 후 `Enter` 키를 누르면, 프로젝트를 생성할 경로를 지정하는 화면이 나옵니다. 원하는 경로를 지정하고, OK 버튼을 누릅니다.

![command palette 에서 프로젝트 경로 선택하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/10.maven-project-directory.png)

OK 버튼을 누르고 나면, 아래와 같이 자동으로 mvn 명령어가 입력되고, 버전 설정에 대해 물어봅니다.

![Terminal 에서 초기 버전 설정하는 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/11.maven-project-versioning.png)

마찬가지로 버전에 대한 설명은 GPT-4의 설명으로 대체합니다.

>기본적으로 Maven은 프로젝트의 버전을 `1.0-SNAPSHOT`으로 설정합니다. `SNAPSHOT`은 프로젝트가 개발 중이며 아직 최종 릴리스되지 않았음을 나타내는 라벨입니다. 이렇게 하면 개발 과정에서 생성되는 아티팩트를 최종 릴리스 버전과 구분할 수 있습니다.
>
>버전 정보를 그대로 사용하려면 Enter 키를 누르십시오. 새로운 버전 번호를 입력하려면 해당 번호를 입력한 후 Enter 키를 누르십시오. 예를 들어, `1.0.0-SNAPSHOT`이나 `0.1.0-SNAPSHOT`과 같은 버전 번호를 입력할 수 있습니다.
>
>개발이 진행되고 프로젝트가 안정화되면, `SNAPSHOT` 라벨을 제거하고 최종 릴리스 버전으로 업데이트할 수 있습니다. 예를 들어, `1.0.0` 또는 `1.1.0`과 같은 버전으로 변경할 수 있습니다. 이렇게 하면 프로젝트의 릴리스 버전을 명확하게 식별할 수 있습니다.

이렇게 Versioning 하는걸 Semantic Versioning 이라고 하는걸 책에서 봤는데, 궁금하신 분들은 찾아보시는 것도 좋을 것 같습니다.

예제에서는 일단 `Enter` 키를 누르고 넘어가겠습니다.

그러면 다시 또 설정한게 맞는지 물어봅니다. 이상이 없다면 Y키를 입력해도되고, 안하고 그냥 `Enter` 키를 눌러도 이상 없이 넘어갑니다.

`Enter` 키를 누르고나면 자동으로 프로젝트 구성이 완료되고, 또 Enter 키를 누르면 Terminal이 종료됩니다.

![Terminal 에서 설정 완료한 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/12.maven-project-setting-done.png)

마지막으로 오른쪽 하단의 팝업 창에서 Open 버튼을 누르면 생성한 프로젝트만의 별도 화면이 실행됩니다.

![Maven 프로젝트 생성 완료 후 실행된 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/13.maven-project-create-complete.png)

### 팝업에서 Open 버튼을 누르지 않은 경우

VS Code를 자주 사용하시던 분들은 당연히 아시겠지만, 스크린샷 찍어 놓은 김에 별도로 프로젝트 Open 하는 방법도 같이 첨부합니다.

먼저 `File` 메뉴를 선택하고 `Open Folder` 를 선택합니다.

![VS Code 에서 File 메뉴 선택한 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/14.Open-Folder.png)

메뉴 선택 후 Command Palette에서 경로를 선택할 수 있습니다. 생성한 프로젝트를 선택하고, OK 버튼을 누릅니다.

![VS Code 에서 Open Folder 선택 후 시연된 Command Palette 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/15.open-folder-command-palette.png)

OK 버튼을 누르고나면, 새로운 창에서 VS Code가 선택한 프로젝트의 경로로 실행됩니다.

### 설정 변경하기

프로젝트를 생성했으면 설정을 마무리해주어야 합니다. 필요한 dependencies 가 있다면 필요에 따라 추가하시면 됩니다. 저는 JDK 버전 설정하는 것만 작성해 보겠습니다.

아래와 같이 pom.xml 파일을 여시고, `<properties>` 의 `<maven.compiler.source>` 와 `<maven.compiler.target>` 을 사용하는 JDK 버전으로  수정합니다.

기본 설정은 아래와 같이 1.7로 설정되어 있고, 제가 기본적으로 사용하는 JDK는 11이라 아래쪽에 알림이 뜨는 것을 볼 수 있습니다.

![pom.xml 파일을 연 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/16.edit-pom.png)

저는 11이 기본으로 사용되므로, 일단 11로 아래와 같이 수정합니다. 

![pom.xml 파일에서 JDK 버전을 수정한 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/17.edit-pom-complete.png)

수정 후 단축키 `ctrl + s` 를 눌러 저장하고 나면, 오른쪽 하단에 동기화 할 것인지 물어보면 `Yes` 를 선택하여 동기화합니다.

![pom.xml 파일에서 JDK 버전을 수정 후 저장 시 시연되는 팝업창](/assets/img/2023-05-13-create-maven-project-in-vscode/18.build-synchronize.png)

### 여러 JDK 버전을 사용하는 경우

저는 JDK 11과 17을 모두 설치해놨습니다. 그래서 17을 사용하고 싶은 경우에는 설정을 변경해야 합니다.

![Ubuntu에서 설치된 JDK 목록을 출력한 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/19.jdk-list.png)

방법은 앞서 pom.xml 파일에서 JDK 11로 수정했던 것 처럼 11을 17로 변경하면 됩니다. 따로 스크린샷 첨부하지는 않겠습니다.

---

아래처럼 VS Code 의 설정 파일인 settings.json 을 수정해주기도 하는데, 실행 시 사용되는 JDK는 pom.xml 이 우선하는 것을 실험(settings.json과 pom.xml 버전을 다르게 설정 후 프로젝트 실행)을 통해 확인했습니다.

![VS Code 에서 settings.json 파일을 연 화면](/assets/img/2023-05-13-create-maven-project-in-vscode/20.settings.json.png)

VS Code Extension인 Language support for Java ™ for Visual Studio Code 가 settings.json에서 제대로 설정 안하면 오류가나는 경우가 있어서 필요하신 분들을 위해 아래에 첨부합니다.

오류를 다시 만들어보려고 해도 똑같은 오류가 발생하지 않아서 발생 조건을 찾지는 못했습니다.

```json
{
    "java.configuration.updateBuildConfiguration": "interactive",
    "java.configuration.runtimes": [
        {
            "name": "JavaSE-11",
            "path": "/usr/lib/jvm/java-11-openjdk-amd64",
        },
        {
            "name": "JavaSE-17",
            "path": "/usr/lib/jvm/java-17-openjdk-amd64",
            "default": true
        }
    ],
    "java.jdt.ls.java.home": "/usr/lib/jvm/java-17-openjdk-amd64",
}
```

WSL을 사용하다보니 path 설정하는 거에서 조금 고생하기도 했었는데, WSL 사용하시는 분들은 참고하셔도 좋을 것 같습니다.

각 설정 항목에 대한 자세한 내용은 해당 Extension 의 GitHub([링크](https://github.com/redhat-developer/vscode-java))을 참조하시면 됩니다.

간단하게 위에 설정한 정도의 JDK 설정에 대한 것만 보시려면 Wiki([링크](https://github.com/redhat-developer/vscode-java/wiki/JDK-Requirements))를 참조하시면 됩니다.

## settings.json 의 설정 둘러보기

그래도 GPT-4 를 이용해 답변을 받아놓은게 있어 참고삼아 붙여넣습니다.

### java.configuration.updateBuildConfiguration

>`settings.json` 파일에서 `"java.configuration.updateBuildConfiguration": "interactive"` 설정은 Java 프로젝트의 빌드 구성이 변경되었을 때 Java Language Server의 동작을 정의하는 옵션입니다.
>`"java.configuration.updateBuildConfiguration"` 설정에는 다음 세 가지 값이 있습니다:
>1. `"disabled"`: 자동으로 빌드 구성을 업데이트하지 않습니다. 이 경우, 개발자가 수동으로 빌드 구성을 관리해야 합니다.
>2. `"interactive"`: 빌드 구성이 변경되면, 개발자에게 변경 사항을 적용할지 묻는 팝업 창이 표시됩니다. 이 옵션을 선택하면, Language Server가 변경 사항을 자동으로 적용하기 전에 개발자의 확인을 받습니다.
>3. `"automatic"`: 빌드 구성이 변경되면, 자동으로 변경 사항을 적용합니다. 이 경우, 개발자에게 묻지 않고 바로 변경 사항을 적용하기 때문에 빌드 구성이 변경될 때마다 프로젝트의 빌드 구성을 업데이트합니다.
>`"java.configuration.updateBuildConfiguration": "interactive"`를 사용하면, 빌드 구성 변경 사항이 발생할 때마다 개발자의 확인을 거치기 때문에 변경 사항에 대한 통제를 더 할 수 있습니다. 하지만 자동으로 빌드 구성을 업데이트하고 싶다면 `"automatic"`을 사용할 수 있습니다.

### default: true

> default: true는 해당 Java 런타임을 프로젝트의 기본 Java 런타임으로 사용하도록 지정하는 것입니다. 여러 버전의 Java 런타임이 시스템에 설치되어 있을 경우, 이 옵션을 사용하여 특정 런타임을 기본으로 설정할 수 있습니다.

### java.jdt.ls.java.home

> `"java.jdt.ls.java.home": "/usr/lib/jvm/java-17-openjdk-amd64"` 설정은 Java Language Server를 실행하는 데 사용되는 Java Runtime Environment(JRE) 또는 Java Development Kit(JDK)의 경로를 지정하는 설정입니다.
>  
> 여기서 "java.jdt.ls.java.home"은 Eclipse JDT(Language Server for Java)의 홈 디렉토리를 가리키며, 이 경로에는 필요한 Java 런타임이 포함되어 있어야 합니다.
> 
> 이 설정은 특히 여러 버전의 Java가 시스템에 설치되어 있는 경우에 중요합니다. 이 설정을 통해 Java Language Server가 어떤 Java 런타임을 사용할지 명확하게 지정할 수 있습니다. 이는 프로젝트의 Java 버전과 Language Server의 Java 버전 사이에 일관성을 유지하고, 예기치 않은 문제를 방지하는 데 도움이 됩니다.
>
>따라서 위의 설정에서 "java.jdt.ls.java.home" 값으로 "/usr/lib/jvm/java-17-openjdk-amd64"를 지정함으로써, Language Server가 Java 17 런타임을 사용하도록 설정하였습니다. 이는 "/usr/lib/jvm/java-17-openjdk-amd64" 경로에 Java 17 버전의 JRE 또는 JDK가 설치되어 있음을 가정하는 것입니다.

## 참고사항

모르는 용어 일부를 GPT-4에 물어봤는데, 맞는지 검증은 안해서 그냥 참고사항으로 붙여넣습니다. 그냥 그런걸 수도 있겠다...? 정도로 보고 넘어가는게 좋겠습니다.

### Java Language Server

Java Language Server는 Java 프로그래밍 언어에 대한 지능형 코드 완성, 오류 검출, 문서화, 리팩터링, 코드 내비게이션 등의 기능을 제공하는 서버 프로그램입니다. 이 서버는 Language Server Protocol (LSP)를 구현하여 에디터와 통신하며, 이를 통해 다양한 코드 에디터와 통합 개발 환경 (IDE)에서 동일한 기능을 제공할 수 있습니다.

Language Server Protocol은 Microsoft가 개발한 프로토콜로, 에디터와 독립적으로 언어 서버를 개발할 수 있게 해줍니다. 이 프로토콜 덕분에, 각각의 에디터에서 동일한 기능을 구현할 필요 없이 언어 서버를 사용하여 코드 지원 기능을 제공할 수 있습니다.

Java Language Server 중 가장 널리 사용되는 것은 Eclipse JDT(Language Server for Java)입니다. 이는 Eclipse IDE에 포함된 Java 개발 도구 (Java Development Tools, JDT)를 기반으로 만들어진 서버로서, Visual Studio Code, Eclipse Che, Emacs, Vim 등 여러 에디터에서 사용할 수 있습니다.

Java Language Server를 사용하면, 에디터와 프로젝트에 대한 지식을 분리하여, 에디터는 텍스트 편집에 집중하고 언어 서버는 언어에 대한 지식을 관리할 수 있습니다. 이로 인해 개발 환경 구축 및 유지 관리가 더 간편해지고, 개발자는 에디터를 자유롭게 선택하여 작업할 수 있게 됩니다.

### Eclipse JDT

Eclipse JDT(Java Development Tools)는 Eclipse 프로젝트의 일부로서, Java 프로그래밍 언어에 대한 통합 개발 환경(IDE) 기능을 제공하는 도구 모음입니다. 이는 Java 코드 작성, 디버깅, 리팩터링, 테스트, 그리고 문서화를 지원하기 위한 다양한 기능을 포함하며, 이러한 기능들을 사용하여 개발자들이 효율적으로 Java 기반의 프로젝트를 개발할 수 있습니다.

Eclipse JDT는 다음과 같은 주요 기능을 제공합니다:

1. **코드 편집**: 지능형 코드 완성, 문법 강조, 자동 정렬 및 서식 지정 등의 기능을 통해 코드 편집을 쉽고 빠르게 할 수 있습니다.
2. **코드 내비게이션**: 클래스, 메서드, 변수로 빠르게 이동할 수 있는 기능을 제공합니다. 또한, 타입 계층 구조를 확인하거나 호출 계층 구조를 탐색할 수 있습니다.
3. **리팩터링**: 코드를 변경하거나 개선하기 위한 자동화된 리팩터링 도구를 포함합니다. 예를 들어, 클래스 이름을 변경하거나 메서드를 추출하는 등의 작업을 수행할 수 있습니다.
4. **오류 검출**: Java 소스 코드의 구문 오류 및 논리 오류를 실시간으로 검출하고 수정 제안을 제공합니다.
5. **디버깅**: 소스 코드를 단계별로 실행하며 변수 값, 스택 추적, 브레이크 포인트 등을 관찰할 수 있는 강력한 디버깅 도구를 제공합니다.
6. **테스트**: JUnit 및 TestNG 테스트 프레임워크와의 통합을 지원하여, 테스트 작성 및 실행을 손쉽게 할 수 있습니다.
7. **문서화**: JavaDoc 도구와 통합되어 코드에 주석을 추가하고 문서를 생성하는 것을 지원합니다.

Eclipse JDT는 Eclipse IDE 내에서 동작하며, Java 개발자들에게 필요한 다양한 기능을 제공하는 통합 개발 환경을 구성합니다. 또한, 이러한 기능을 Language Server Protocol을 통해 다른 에디터와 IDE에서도 사용할 수 있도록 Java Language Server로 제공하고 있습니다. 이를 통해 개발자들은 자신이 선호하는 에디터에서도 Eclipse JDT의 기능을 활용할 수 있습니다.

## Outro

Java 개발할때는 결국 Intelli J나 Eclipse를 사용하게 되겠지만, VS Code를 제일 선호하다 보니 글을 작성해 봤습니다.

도움이 되셨길 바라봅니다.
