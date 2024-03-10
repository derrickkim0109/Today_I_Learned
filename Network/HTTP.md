<h1><center> HTTP </center></h1>

###### tags: `💻 TIL`, `Computer Science`, `Network`, `HTTP`
###### date: `2024-02-01T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [HTTP - 위키백과](https://ko.wikipedia.org/wiki/HTTP)
> [HTTP](https://www.cloudflare.com/ko-kr/learning/ddos/glossary/hypertext-transfer-protocol-http/)

## HTTP

하이퍼텍스트 전송 프로토콜은 월드 와이드 웹의 토대이며 하이텍스트 링크를 사용하여 웹 페이지를 로드하는 데 사용된다. HTTP는 네트워크 장치 간에 정보를 전송하도록 설계된 애플리케이션 계층 프로토콜이며 네트워크 프로토콜 스택의 다른 계층 위에서 실행된다. HTTP를 통한 일반적인 흐름에는 클라이언트 시스템에서 서버에 요청한 다음 서버에서 응답 메시지를 보내는 작업이 포함된다.

### HTTP Method

HTTP Method는 HTTP Request가 쿼리된 서버에서 기대하는 작업을 나타낸다. 가장 일반적인 두 메서드는 GET,POST가 있다. GET은 응답으로 정보를 기대하는 반면, POST요청은 일반적으로 클라이언트가 웹 서버에 정보를 제출하고 있음을 나타낸다.

### HTTP Header

![image](https://hackmd.io/_uploads/Syjp4gq5a.png)

> Google Chrome의 네트워크 탭에 있는 HTTP Request Header 예시

HTTP Header에는 키와 값에 저장된 텍스트 정보가 포함되어 있으며, 헤더는 모든 HTTP Request(or Response)에 포함된다. 이러한 헤더는 클라이언트가 사용하는 브라우저 및 요청되는 데이터와 같은 핵심 정보를 전달한다.

### HTTP Request에는 무엇이 들어있나?

HTTP Request는 웹 브라우저와 같은 인터넷 통신 플랫폼에서 웹 사이트를 로드하는 데 필요한 정보를 Request하는 방법이다. 

인터넷을 통해 이루어진 각 HTTP요청은 서로 다른 유형의 정보를 전달하는 일련의 인코딩된 데이터를 전달한다. 일반적인 HTTP요청에는 다음이 포함된다.

1. HTTP 버전 유형
2. URL
3. HTTP Method
4. HTTP Request Header
5. (Optional) HTTP Body

### HTTP Request Body에는 무엇이 있을까?

Request의 Body는 Request에서 전송되는 정보의 Body를 포함하는 부분이다. HTTP Request의 본문에는 사용자 이름 및 비밀번호 또는 양식에 입력된 기타 데이터와 같이 웹 서버에 제출되는 모든 정보가 포함된다. 

### HTTP Response에는 무엇이 있나?

HTTP Response는 웹 클라이언트(종종 브라우저)에서 HTTP Request에 대한 Response로 인터넷 서버로부터 수신하는 Response이다. 

HTTP Response에는 다음이 포함된다. 

- HTTP Status code
- HTTP Response Header
- (Optional) HTTP Body

### HTTP Response Header

HTTP Request와 마찬가지로 HTTP Response에는 응답 본문에서 전송되는 데이터의 언어 및 형식과 같은 중요한 정보를 전달하는 헤더가 함께 제공된다.

![image](https://hackmd.io/_uploads/BJrc5g55a.png)

> Google Chrome 네트워크 탭에 있는 HTTP Response Header의 예시

### HTTP Response Body에는 무엇이 있을까?

GET 요청에 대한 성공적인 HTTP Response에는 일반적으로 요청된 정보가 포함된 Body가 있다. 대부분의 웹 요청의 경우 이는 웹 브라우저에서 웹 페이지로 변환되는 HTML 데이터다.

### HTTP Status Code

HTTP 상태 코드는 HTTP Request가 성공적으로 완료되었는지 여부를 나타내는 데 가장 자주 사용되는 3자리 코드이다. 상태 코드는 5개의 블록으로 나뉜다.

- 1xx informational
- 2xx Success
- 3xx Redirection
- 4xx Client Error
- 5xx Server Error

`xx`는 00~99 사이의 다른 숫자를 나타낸다. 
