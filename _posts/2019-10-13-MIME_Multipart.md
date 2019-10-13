---
layout: post
title: "MIME과 Multipart"
author: "Hongmin Park"
---

웹을 운영하는 사람이면 MIME type에 대해 한 번 쯤은 들어봤을 수 있다. 단순히 확장자라고 생각했는데, 메일을 발송하는 코드를 작성하다가 MIME Message라는 용어와 Multipart라는 게 혼용되어 사용되는 것을 보았다. Multipart라는 것은 파일을 보낼 때 사용되는 어떤 것 이라고만 알고있었는데, 이 모든게 순간 혼란스러워졌다. MIME과 Multipart라는 것 각각의 개념에 대한 이해가 부족했고, 그래서 이 두개가 왜 관계가 있는지 몰랐다. 아래는 내가 메일을 보낼 때 사용했던 코드의 일부이다.(일부 변수들이 제거된 코드이므로 흐름만 봐야한다.) 각 로직의 설명을 주석으로 달아놓았는데, MIME/Multipart에 대해 이해한 후 코드를 읽어보면 왜 이런 로직이 되었는지 이해할 수 있다. 당장 이해가 되지 않는다면, 이어서 나오는 정리부분을 읽어보면 된다. 내용은 [Mozilla Developer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)를 참고하여 이해/정리해보았다.<br>

```java
public class SendMail
{
  public static void sendSuccessMail(JavaMailSenderImpl mailSenderService) throws Exception {

    mailSenderService.testConnection();

    // 메일 발송에 필요한 정보가 담길 MimeMessage 객체와 메시지 본문을 담을 MimeBodyPart 객체를 생성한다.
    MimeMessage msg = mailSenderService.createMimeMessage();
    MimeBodyPart msgBodyPart = new MimeBodyPart();
    
    // MimeMessage 객체에는 수신인, 발송인, 제목 등의 정보를 담는다.
    msg.setFrom(new InternetAddress(FROM, FROMNAME));
    msg.setRecipient(Message.RecipientType.TO, new InternetAddress(TO));
    msg.setSubject(SUBJECT);

    // MimeBodyPart 에는 메시지의 본문을 담을 것인데, 파일과 함께 본문을 발송할 것이므로 MimeMultipart 객체가 필요한다. 
    // MimeMultipart에는 여러 content-type의 MimeBodyPart를 담을 수 있고, 최종적으로 Message의 본문으로 담긴다. 
    MimeMultipart mimeMultipart = new MimeMultipart();

    // MimeMultipart 객체에 우선 html형식의 BODY 하나를 넣는다. 
    msgBodyPart.setContent(BODY, "text/html; charset=UTF-8;");
    mimeMultipart.addBodyPart(msgBodyPart);

    // MimeMultipart 객체에 파일이 첨부된 MimeBodyPart를 하나 더 생성하여 넣는다. 
    MimeBodyPart attatchBodyPart = new MimeBodyPart();
    DataSource source = new FileDataSource(reportZipFilePath);
    attatchBodyPart.setDataHandler(new DataHandler(source));
    attatchBodyPart.setFileName(String.valueOf(dumpFileName) + "_report.zip");
    mimeMultipart.addBodyPart(attatchBodyPart);
    
    // MimeBodyPart들이 담긴 MimeMultipart를 Message의 본문으로 지정하고 메일을 발송한다. 
    msg.setContent(mimeMultipart);
    mailSenderService.send(msg);
  }
```

## MIME 이란?
**Multipurpose Internet Mail Extension, MIME**는 doument, file, byte codes 등이 어떤 형식의 데이터인지를 암시하는 규격이다. 이름에서도 알 수 있듯이 다용도 **메일 표현 규격**이다. 기존에 SMTP을 위한 인터넷 표준이었는데, HTTP 통신에서도 SMTP와 비슷한 요청과정이 일어나서 MIME Type을 사용하게 된다. 우리는 지금 HTTP가 더 익숙하지만 역사 속에서는 SMTP가 형님이다. [SMTP는 1981년에 처음 정의](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol)되었고, [HTTP는 1989년에 처음 시작](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)되었다. 그래서 메일을 작성할 때에도 MIME Message라고 하고, HTTP에서도 MIME Type을 사용한다. 웹에서의 MIME type은 파일의 확장자를 뜻하는 것이 아니라, 웹 엔진이 URL에 의해 요청을 어떻게 처리해야할지를 결정하는 규격이다. 

### 구조
> type/subtype

구조로 되어있다. 예를 들어 `text/plain`, `text/html`와 같이 사용된다. '/'를 기준으로 좌측에 있는 **type**에는 **discrete**과 **multipart**라는 두 종류가 있다. 
#### discrete
하나의 텍스트, 음악, 영상 파일 처럼 single 구조의 타입이다. `application, audio, example, font, image, video` 등이 있다.

#### multipart
여러 형태의 component들로 이루어져 있는 document를 뜻한다. 예를 들어 각자 MIME type이 있는 여러 파일/document를 한 번에 보낼 때에 이 요청의 MIME type은 **multipart**이다. HTML Form에서 POST 메소드에 쓰이는 `multipart/form-data`나 `multipart/byteranges`를 제외하고는 multipart type은 discrete type과 동일하게 처리된다. 브라우저는 multipart 요청을 받은 후에 그에 맞게 화면에 뿌려준다(정의되어 있지 않은 multipart는 'Save As'창이 뜬다.) 주로 각자 자신의 MIME type을 가진 요소들을 하나의 데이터로 보낼 때에 multipart 타입이 사용된다. 
<br> 예를 들어 아래와 같은 `multipart/form-data`를 보자.(example from [here](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types))

```html
<form action="http://localhost:8000/" method="post" enctype="multipart/form-data">
  <input type="text" name="myTextField">
  <input type="checkbox" name="myCheckBox">Check</input>
  <input type="file" name="myFile">
  <button>Send the file</button>
</form>
```

위와 같은 multipart/form-data는 실제 아래와 같은 HTTP 메시지 형태로 전송된다.

```
POST / HTTP/1.1
Host: localhost:8000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------8721656041911415653955004498
Content-Length: 465

-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myTextField"

Test
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myCheckBox"

on
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myFile"; filename="test.txt"
Content-Type: text/plain

Simple file.
-----------------------------8721656041911415653955004498--
```
**Boundary**를 기준으로 다양한 Content-Type의 데이터가 각각 전송되는 것을 확인할 수 있다.
<br><br>
정리해보면, **MIME**은 파일을 어떻게 처리할 것에 대한 규격이고(파일 확장자와는 엄연히 다르지만 느낌은 비슷하다. 확장자는 무언의 약속, MIME은 유언의 약속), **Multipart**는 여러 MIME type 중 한 종류로서 여러 데이터를 하나의 request로 보낼 때에 '지금 보낸 하나의 데이터에 여러 형태의 데이터가 있다'를 전달해 주는 방식이라고 생각할 수 있다. <br>
이렇게 MIME과 Multipart에 대해 간단히 정리해보았는데, 이제 처음부분의 메일발송 코드를 다시 읽어보자. 이제 이해가 갈 것이다.
