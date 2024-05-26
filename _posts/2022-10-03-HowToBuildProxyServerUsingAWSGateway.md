---
layout: post
title: "AWS Gateway를 이용한 프록시 서버 구축하기"
excerpt: ""
subtitle: "AWS Gateway"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-11-16
tags: [AWS]
---

### 목차

### Proxy 서버 구축 배경

- 공식 홈페이지 - Contact 페이지 내 회사 소개서를 다운로드 시, 입력된 고객사 정보가 Notion 페이지에 자동 저장되는 DB 기능을 구현하려 합니다. 

![homepage.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQNERqJ7LjYdTJ546wHbpeoRAT75S0nPkUTwauzDeF77jRg?width=660)

![Untitled](https://1drv.ms/i/c/cdf0612ba3befd25/IQMMQkpvGUELSZev2gDnxipvATNfpsMn-Ny33FlCFon76cg?width=660)

- 웹 페이지에서 axios 라이브러리를 활용하여, 직접 노션 서버에 데이터를 보내는 형식으로 구현하려 했으나 CORS 에러가 발생했습니다. 
- 노션 서버에서 응답을 보낼 때 Header에 Access-Control-Allow-Origin 값을 변경하면 되지만 노션 서버 조작 불가 → Proxy 서버를 이용하여 서버 간 통신으로 CORS 우회 가능한 점을 알 수 있습니다.
- 또한, HTTP 통신 시, 노션 API를 이용하는데 필요한 페이지 ID값과 토큰 정보가 그대로 노출되는 취약점 존재 → 민감한 정보는 Proxy 서버에 저장하여 노션 API에 맞게 매핑하면 노출을 막을 수 있습니다. 
- CI/CD 구축 시 도입한 AWS Gateway를 이용하여 Proxy 서버를 구축하기로 결정하였습니다! 

### **Notion API 세팅**

- 클라이언트에서 노션 서버에 데이터를 요청하려면 인증키가 필요한데, Notion API에서 우리가 원하는 워크스페이스에 대한 접근 권한을 설정해줍시다.
- 노션 개발자 사이트에 접속 - 로그인 후 오른쪽 최상단에 My integrations 페이지 접속합니다.

[Start building with the Notion API](https://developers.notion.com/)

- New integration 생성 버튼을 클릭

![Untitled1](https://1drv.ms/i/c/cdf0612ba3befd25/IQPgDt-_VxFERqjE3NczOVFYAYQGc8EmWAtONYiDNBR7PfA?width=432&height=185)

- Secrets은 내부용 통합 토큰으로써 워크 스페이스에 접근을 가능케 합니다. 노션 서버에 API를 보낼 때, header에 들어가게 될 값이니 메모 필수!
- Integration type은 다른 노션 유저까지 사용할 수 있게 공개하려면 Public integration을 선택하시면 되요.
- Capabilities 에서는 워크스페이스에 등록된 유저들의 권한을 설정해줍시다. 

![Untitled2](https://1drv.ms/i/c/cdf0612ba3befd25/IQOzVAayDJJWTLEfNWSPiVa_AS2_s29eSJZZhXiyepAgYR4?height=660)

![Untitled3](https://1drv.ms/i/c/cdf0612ba3befd25/IQOzVAayDJJWTLEfNWSPiVa_AS2_s29eSJZZhXiyepAgYR4?height=660)

- 노션 페이지 생성하기 클릭 - database에 table 항목을 누르면 표가 삽입된 페이지가 생성됩니다.
- 표 머리글 행에 있는 값들은 querystring 로 인식 ex) 이름=Sasha&회사=Tesla...
- 노란색 음영 처리된 url은 해당 노션 페이지의 ID 값이며, 이후 body에 들어갈 정보니 메모해둡시다.

![Untitled4](https://1drv.ms/i/c/cdf0612ba3befd25/IQO9OwmefOboTaW1GBw5cvhHAfyZpgpS1210osczIdqZBbQ?width=660)

- 작성한 integration을 노션 페이지에 연결해줘야 수집 기능을 활성화 시킬 수 있습니다.
페이지 오른쪽 상단에 share - 검색창 - 해당 integration 클릭!

![Untitled5](https://1drv.ms/i/c/cdf0612ba3befd25/IQMujz_U3eaaRofX4bEBfc1iATMzSttVlSSUDsu6qTtApos?width=660)

---

### AWS Gateway 세팅 ① - Lambda 이용 X

- Gateway 내 데이터 처리 로직은 다음과 같습니다. 

![Untitled6](https://1drv.ms/i/c/cdf0612ba3befd25/IQPAgXZ-14UmSYTdyiT7Zg2IAREX95lcCs4fLX4naXnpEnw?width=660)

① 메서드 요청: 클라이언트에서 노션에 저장할 데이터를 querystring 형식으로 본 게이트웨이 엔드 포인트로 전달

② 통합 요청: 노션 서버로 보낼 API를 정의해줍시다.

- 엔드포인트 URL에는 노션 API url을 기재하시면 되요.

![Untitled7](https://1drv.ms/i/c/cdf0612ba3befd25/IQOPu1-ZPhJwRo7mhwkS-aOzAciixcx6gnhfaTzq9bzA4D8?width=660)

- HTTP 헤더와 매핑 템플릿에 각각의 정보 입력이 필요한데요,
- 먼저, HTTP 헤더 내 Notion-Version에는 ‘2021-08-16’ 값을 기입하며 Authorization에는 문자열 Bearer 뒤에 Notion API 생성 시 메모한 토큰 값을 기재해줘요.

![Untitled8](https://1drv.ms/i/c/cdf0612ba3befd25/IQN9fs25XdpJToAtq-1E1h7RAcYLn4L53qRoOge5Uu-QoXA?width=660)

<aside>
💡 값 입력 시 홑따옴표로 꼭 묶어주는거 잊지 맙시다! 
</aside>

- content type은 application/json
- 매핑 템플릿에는 메서드 요청에서 받은 querystring 데이터와 메모한 노션 페이지 ID를 아래와 같이 기입해줍시다.

  ```json
  {"parent":{"database_id": "notion_page_id"},
  "properties":
      {"회사명": { "title": [ { "text": { "content": "$method.request.querystring.company" } } ] },
      "연락처": { "rich_text": [ { "text": { "content": "$method.request.querystring.mobile" } } ] },
      "담당자": { "rich_text": [ { "text": { "content": "$method.request.querystring.name" } } ] },
      "직책": { "rich_text": [ { "text": { "content": "$method.request.querystring.position" } } ] },
      "이메일": { "url": "$method.request.querystring.email" }
  }}
  ```

③ 클라이언트 - 테스트 페이지에서 API가 제대로 작동하는지 확인이 가능합니다. 

- 상기 노션 테이블 생성 시 설정한 **표 머리글 행에 있는 값(ex. company, name, mobile ...)** 쿼리 문자열에 삽입해줍시다~
- 노션 페이지 내 테이블에 테스트한 값이 제대로 들어갔는지 확인 필수!

![Untitled9](https://1drv.ms/i/c/cdf0612ba3befd25/IQPOZ5iuRKeQSoNGCYzTCovsAaGhyMJ1nzO-Mj1luhm0Mok?width=660)

④ 테스트 통과되면 작업 탭 내 CORS 활성화 및 API 배포를 진행합니다.

---

### AWS Gateway 세팅 ② - Lambda 이용
    
- Gateway 만을 이용하는 ①번의 방법으로 기능을 구현했지만, Access-Control-Allow-Origin 값이 * 로 (wildcard, 모든 origin에 대해 리소스 공유 허용) 설정되는 한계가 존재합니다.

- wildcard일 경우, HTTP 인증 정보 (ex. cookie.. ) 를 사용할 수 없게 됩니다. 따라서 원하는 특정 도메인을 origin으로 설정해보죠.
    
    > **교차 출처 리소스 공유 (CORS)**
    **링크:** [https://developer.mozilla.org/ko/docs/Web/HTTP/CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)

- AWS Gateway에서 지원하는 CORS 활성화 기능 관련하여 한 개의 도메인만 인식 가능한 상태에요. 😟😟😟
- 도메인에 따라서 api를 생성할 수 있지만, 비효율적이며 멀티 도메인을 인식할 수 있게 한다면 간단하게 해결할 수 있네요. → AWS Lambda를 활용하여 멀티 도메인을 인식하게 만드는 로직을 추가해줍시다.
- AWS Lambda - 새 함수 생성, 런타임은 Node.js, Python, Java 등 다양한 환경을 제공하는 끝내주는 기능입니다!

![Untitled10](https://1drv.ms/i/c/cdf0612ba3befd25/IQN1xmMUq9E-So56rEuM96nqAdqlz9A-sVbb5TBeC4dbPxU?width=660)

- 노란색 원 부분에 하기의 코드 스니펫을 삽입 후 Deploy 버튼을 눌러서 저장해줍시다.

![Untitled11](https://1drv.ms/i/c/cdf0612ba3befd25/IQPfszPKY4J2S5AJgJ3tw7ZeAfeb-GMK22lVqsHaYTN_-YI?width=660)

- 코드는 크게 request의 origin url이 whitelist에 포함되어 있는지 여부를 판별하는 로직과 노션 서버로 데이터를 가공해서 보내는 로직 두 가지로 구성

  ```javascript
  const https = require('https')
  // 허용된 origin whitelist
  const allowedOrigins = [
      https://SashaInSPb.co.kr
  ];
  
  // 데이터를 가공해서 notion에 http 요청을 보내는 함수
  const doPostRequest = (params) => {
      const data = {
      parent: { database_id: '페이지 id를 삽입하세요' },
      properties: {
        회사명: { title: [{ text: { content: params.company } }] },
        연락처: { rich_text: [{ text: { content: params.mobile } }] },
        담당자: { rich_text: [{ text: { content: params.name } }] },
        직책: { rich_text: [{ text: { content: params.position } }] },
        이메일: { rich_text: [{ text: { content: params.email } }] },
      }
      }
  
      return new Promise((resolve, reject) => {
        const options = {
          hostname: 'api.notion.com',
          port: 443,
          path: '/v1/pages',
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Notion-Version': '2021-08-16',
            'Authorization': 'Bearer 노션 API 토큰을 입력하세요'
          }
        };
  
      const req = https.request(options, (res) => {
        resolve(JSON.stringify(res.statusCode));
      })
  
      req.on('error', (e) => {
        reject(e.message);
      });
  
      req.write(JSON.stringify(data));
      req.end();
      })
  }
  
  // 람다함수 핸들러
  exports.handler = async (event, context) => {
      const origin = event.headers.Origin || event.headers.origin;
      const params = event.queryStringParameters;
      
      await doPostRequest(params)
      .then(result => console.log(`Status code: ${result}`))
      .catch(err => console.log(`Error doing the request for the event: ${JSON.stringify(event)} => ${err}`));
      
      let goodOrigin = false;
  
      if (origin) { // whitelist 내에 있는 url일 경우 goodOrigin은 true
          allowedOrigins.forEach(allowedOrigin => {
              if (!goodOrigin && origin.match(allowedOrigin)) {
                  goodOrigin = true;
              }
          });
      }
      
      context.succeed({
          headers: {
              "Access-Control-Allow-Headers": "Accept,Accept-Language,Content-Language,Content-Type,Authorization,x-correlation-id",
              "Access-Control-Expose-Headers": "x-my-header-out",
              "Access-Control-Allow-Methods": "DELETE,GET,HEAD,OPTIONS,PATCH,POST,PUT",
              "Access-Control-Allow-Origin": goodOrigin ? origin : null, // whitelist 이외의 origin의 경우 null로 반환
              "X-key": "check"
          },
          statusCode: 200
      });
  };
  ```

- 테스트에서 간단히 데이터를 보내 함수가 작동하는지 확인 가능하며, 모니터링에 CloudWatch에서 로그 보기를 통해 제한적으로 디버깅이 가능합니다.

![Untitled12](https://1drv.ms/i/c/cdf0612ba3befd25/IQN6q4roM_D2TJRxknQY6T5uAbK6y0vGzJxTvdfF12BJ3ZM?width=660)

- 트리거는 작성한 lambda 함수를 작동시키기 위한 기능으로 본 프로젝트의 경우, Gateway에서 API call을 받는 단계가 이에 해당. 아래와 같이 함수와 연결될 게이트웨이를 새로 생성

![Untitled13](https://1drv.ms/i/c/cdf0612ba3befd25/IQN3cz2JogXoS4ghQuG4nXxKAQYbxMys1Yt9YVaEvZkZCXM?height=660)

- Gateway 내 데이터 처리 로직은 다음과 같음

![Untitled14](https://1drv.ms/i/c/cdf0612ba3befd25/IQMgJFahVZAsSrNsv5kwJQ3-Ab2NA69o8VskkSqKNzxnLqw?width=660)

① 메서드 요청: 클라이언트에서 노션에 저장할 데이터를 querystring 형식으로 본 게이트웨이 
엔드 포인트로 전달

- URL 쿼리 문자열 파라미터에 전달하고자 하는 데이터를 쿼리문자열로 넣어주자

② 통합 요청:  Lambda proxy 통합 이용을 컨펌하는 단계

- 통합 유형은 Lambda 함수로 넣고, 프록시 통합 사용에 체크

![Untitled15](https://1drv.ms/i/c/cdf0612ba3befd25/IQN9EiCXN48CR5xWeBZI21uUATECqKky0cErq-JUQDfOT_E?width=660)

③ API가 잘 작동하는지 임시 테스트 진행 

[https://www.notion.so/2bytes/AWS-Gateway-91c50e3fbfeb4e748bf7169c7c2f996e#68c8e274748b4318bd71467bf2d0d178](https://www.notion.so/AWS-Gateway-91c50e3fbfeb4e748bf7169c7c2f996e?pvs=21)

④ 테스트가 완료되었다면 CORS 활성화 및 API 배포 진행

---

### 추가 보완점 (Technical debt)

- 보안 탭에 AWS IAM 서비스를 이용하거나 API KEY를 이용하여 API 엔드포인트에 대한 보안 메커니즘을 강화할 수 있음. 현재는 ‘열기’로 설정되어 엔드 포인트에 대해 누구나 접근 가능한 상태

### Ref.

[API Gateway에서 Lambda 프록시 통합 설정](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html)

[AWS Lambda를 이용해서 HTTP API 만들기 #2 :: Outsider's Dev Story](https://blog.outsider.ne.kr/1206)

[Dev: Notion API 시작부터 첫 요청까지(Web)](https://medium.com/hcleedev/dev-notion-api-시작부터-첫-요청까지-web-6a2307b477c8)

[AWS API Gateway와 Lambda를 활용해 REST API 구축하기 ②](https://ndb796.tistory.com/304?category=1045560)

[AWS API Gateway - CORS "access-control-allow-origin" - multiple entries](https://stackoverflow.com/questions/39628640/aws-api-gateway-cors-access-control-allow-origin-multiple-entries/41708323#41708323)

[AWS Lambda HTTP POST Request (Node.js)](https://stackoverflow.com/questions/47404325/aws-lambda-http-post-request-node-js)
