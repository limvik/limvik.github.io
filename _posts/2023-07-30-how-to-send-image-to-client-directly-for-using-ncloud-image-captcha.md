---
layout: post
title: NCloud Image CAPTCHA API 사용 시 클라이언트에 이미지 바로 보내는 방법
categories:
- etc
tags:
- NCloud
- Base64
- CAPTCHA
- Image
date: 2023-07-30 15:08 +0900
---
## Intro

교육 과정에서 naver ncloud를 지원해줘서, 팀 프로젝트에 회원가입용으로 Naver Image CAPTCHA API([링크](https://www.ncloud.com/product/applicationService/captcha)) 를 사용해봤습니다. CAPTCHA Image를 서버에 저장할 이유가 없어 저장하지 않고 클라이언트로 보낼 방법을 찾다가, Base64로 인코딩한 문자열을 \<img> 태그의 `src` 속성에 직접 지정하는 방법을 알게되어 기록 및 공유 겸 작성합니다.

![회원가입 화면](/assets/img/2023-07-30-how-to-send-image-to-client-directly-for-using-ncloud-image-captcha/01-signup.png)

## API 가이드 예제 활용하기

처음에는 API 가이드([링크](https://api.ncloud-docs.com/docs/ai-naver-captcha-image))에 있는 예제를 그대로 사용해서, API 요청을 통해 받은 CAPTCHA 이미지를 서버에 저장한 후 해당 이미지를 불러와서 사용했습니다.

Spring 을 사용하고 있어서 Service, Controller 등이 등장합니다.

```java
// NaverImageCaptchaService.java
public String getCaptchaImagePath(String captchaKey) {
    try {
        String apiURL = "https://naveropenapi.apigw.ntruss.com/captcha-bin/v1/ncaptcha?key=" + captchaKey + "&X-NCP-APIGW-API-KEY-ID=" + clientId;
        URL url = new URL(apiURL);
        HttpURLConnection con = (HttpURLConnection)url.openConnection();
        con.setRequestMethod("GET");
        int responseCode = con.getResponseCode();
        BufferedReader br;
        if(responseCode==200) {
            String captchaImageFileName = Long.valueOf(new Date().getTime()).toString();
            File f = new File("src/main/resources/static/images/captcha/" + captchaImageFileName + ".jpg");
            f.createNewFile();

            InputStream is = con.getInputStream();
            int read = 0;
            byte[] bytes = new byte[1024];
            OutputStream outputStream = new FileOutputStream(f);
            while ((read = is.read(bytes)) != -1) {
                outputStream.write(bytes, 0, read);
            }
            is.close();
            return f.getPath();
        } else {
            br = new BufferedReader(new InputStreamReader(con.getErrorStream()));
            String inputLine;
            StringBuffer response = new StringBuffer();
            while ((inputLine = br.readLine()) != null) {
                response.append(inputLine);
            }
            br.close();
            log.info("[Failed Create CAPTCHA Image] CAPTCHA Image Request Key:" + captchaKey +
                    "Message: " + response.toString());
        }
    } catch (Exception e) {
        log.info(e.getMessage());
    }

    return "";

}
```

### 발생한 문제

이미지 파일을 저장했으니, thymeleaf 를 쓰고 있어 불러올 이미지 경로를 src 속성에 아래와 같이 지정했습니다.

```html
<img th:src="@{/images/captcha/__${captchaImageFileName}__}" class="img-fluid rounded" alt="Captcha 이미지"/>
```

그런데 동적으로 생성된 이미지 파일이 프로젝트 내부 경로에 있으면, 클라이언트에서 인식되지 않았습니다.

이미지 저장경로를 프로젝트 외부 경로에 만들까 고민하다가, 클라이언트로 이미지를 바로 보내버릴 방법은 없는지 찾아보게 됐습니다. 그리고 GPT를 통해 쉽게 찾을 수 있었습니다.

## Base64 로 이미지를 인코딩해서 사용하기

GPT 를 통해 얻은 정보를 바탕으로, 일단 저장하는 것은 그대로 놔두고, 저장한 경로에 있는 이미지를 byte로 불러와 Base64 인코딩하여 반환했습니다.

Controller 의 관련 코드만 보면 아래와 같습니다.

```java
// Controller
String captchaImagePath = naverImageCaptchaService.getCaptchaImagePath(captchaKey);
model.addAttribute("encodedCaptchaImage", CaptchaUtil.encodeBase64Image(captchaImagePath));

// CaptchaUtil.java
public static String encodeBase64Image(String imagePath) {
    Path path = Paths.get(imagePath);
    byte[] bytes;
    try {
        bytes = Files.readAllBytes(path);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
    return Base64.getEncoder().encodeToString(bytes);
}
```

그리고 html 에서는 img 태그의 src 속성에 인코딩된 문자열 이미지 앞에 `'data:image/jpeg;base64,'` 를 추가하여 줍니다.

```html
<img th:src="@{'data:image/jpeg;base64,' + ${encodedCaptchaImage}}" class="img-fluid rounded" alt="Captcha 이미지"/>
```

그러면 아래와 같이 img 태그의 src 속성에 설정한 Base64로 인코딩된 문자열이 바로 이미지로 시현되는 것을 볼 수 있습니다.

![Base64로 인코딩된 이미지 시현](/assets/img/2023-07-30-how-to-send-image-to-client-directly-for-using-ncloud-image-captcha/02-captcha-image.png)

### API 예제 코드 변경하기

이미지 파일이 이상 없이 시현되는 것을 확인했으니, Service 에서 파일을 저장하지 않도록 수정했습니다.

```java
public byte[] getCaptchaImage(String captchaKey) {
    String apiURL = buildApiURL(captchaKey);
    try {
        HttpURLConnection con = establishHttpConnection(apiURL);
        return handleResponse(con);
    } catch (Exception e) {
        log.info(e.getMessage());
    }

    return new byte[]{};

}

private String buildApiURL(String captchaKey) {
    return "https://naveropenapi.apigw.ntruss.com/captcha-bin/v1/ncaptcha?key="
            + captchaKey + "&X-NCP-APIGW-API-KEY-ID=" + clientId;
}

private HttpURLConnection establishHttpConnection(String apiURL) throws IOException {
    URL url = new URL(apiURL);
    HttpURLConnection con = (HttpURLConnection)url.openConnection();
    con.setRequestMethod("GET");
    return con;
}

private byte[] handleResponse(HttpURLConnection con) throws IOException{
    if(con.getResponseCode() == 200) {
        return con.getInputStream().readAllBytes();
    } else {
        handleError(con);
        return new byte[]{};
    }
}

private void handleError(HttpURLConnection con) throws IOException {
    var br = new BufferedReader(new InputStreamReader(con.getErrorStream()));
    String inputLine;
    StringBuffer response = new StringBuffer();
    while ((inputLine = br.readLine()) != null) {
        response.append(inputLine);
    }
    br.close();
    log.info("[Failed Create CAPTCHA Image] Message: " + response.toString());
}
```

Controller 에서도 Util 클래스 없이 바로 Base64로 인코딩했습니다.

```java
String encodedCaptchaImage = Base64.getEncoder().encodeToString(naverImageCaptchaService.getCaptchaImage(captchaKey));
model.addAttribute("encodedCaptchaImage", encodedCaptchaImage);
```

## 예상 문제 - 메모리

아무래도 파일로 저장한 후에 클라이언트에서 이미지를 요청해서 받아가는 방식이 아니다보니, CAPTCHA 이미지가 처리되는 동안 서버의 메모리를 점유할 수 밖에 없습니다.

그런데 CAPTCHA 이미지는 사이즈가 크지 않기 때문에 큰 문제 없이 사용할 수 있을 것으로 예상됩니다.

저장하는 방식을 바로 바꿨던 것은 아니라 144개의 CAPTCHA 이미지가 쌓여서 용량 순으로 이미지를 정렬해봤습니다.

![탐색기에서 본 CAPTCHA 이미지 저장된 목록](/assets/img/2023-07-30-how-to-send-image-to-client-directly-for-using-ncloud-image-captcha/03-captcha-image-list.png)

가장 큰 이미지의 크기가 10KB 인 것을 확인할 수 있고, 그마저도 몇개 안됩니다.

최악의 경우인 10KB 로 가정하고, 1만 명씩 같은 시간에 동시에 가입한다고 가정하면, 대략 이미지만을 위해 100MB 의 메모리가 필요합니다.

현재 지원받은 서버의 메모리는 2GB 이고, TOMCAT 서버, MySQL 서버, Jenkins 등이 실행 중입니다. 잠시 살펴본 가용 메모리는 거의 600MB 가까이 됩니다.

![cat /proc/meminfo](/assets/img/2023-07-30-how-to-send-image-to-client-directly-for-using-ncloud-image-captcha/04-meminfo.png)

>`cat /proc/meminfo` 명령어로 확인할 수 있습니다.

데이터베이스 읽기/쓰기 작업 등이 동시에 진행되면 가용 메모리가 줄어들기는 하겠지만, 충분히 감당가능하지 않을까 추측해봅니다. (더 정확하게 보려면 성능 테스트 방법 학습이 필요하겠습니다.)

## Outro

CAPTCHA 이미지는 용량이 작아서 별 문제가 없을 것으로 예상되지만, 예상 못한 상황도 있을 수 있으니 Base64로 이미지를 인코딩해서 클라이언트로 보낼 때는 가용 메모리를 확인하면서 사용하는게 좋겠습니다.