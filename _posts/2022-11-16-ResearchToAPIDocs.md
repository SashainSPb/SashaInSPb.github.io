---
layout: post
title: "API docs tool의 종류와 특징에 대해서 알아보자"
excerpt: ""
subtitle: "API docs"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-11-16
tags: [Swagger]
---

# API docs Tool 에 대한 고찰

## 목표

- API docs는 개발자 간 협업 도구로써의 중요한 역할을 맡고 있습니다. 시중에는 다양한 종류의 API docs tool 제품 군이 존재하며,
각각 어떤 특징이 있는지 알아보고 어떤 tool이 가장 적합할지 알아보려고 합니다.

## 리서치 기준

- OpenAPI 지원 여부
- API 문서 작성에서 테스팅까지의 개발 공수
- 편리하고 직관적인 UI 여부

## 소개

1.   **DapperDox**
    
    ![dapperdox.png](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/dapperdox.png)
    
    - 장점
        
        사용하기 쉬움
        
        다양한 종류의 문서 테마 지원 가능
        
    - 단점
        
        비 정기적인 업데이트
        
2. **Postman**
    
    ![postman1.png](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/postman1.png)
    
    - 장점
        
        다양한 API 사례와 기계가 이해할 수 있는 명령을 쉽게 배포 가능
        모든 API docs 자동 업데이트 기능
        
    - 단점
        
        복잡한 UI
        깊은 러닝커브 요구
        

## 소스코드 인터페이스를 가진 API docs Tools

c. **Slate (widdershins)**

![Untitled](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/Untitled.png)

OpenAPI 3.0 / Swagger 2.0 / AsyncAPI 1.x / Semoasa 0.1.0  문서화

d. **Redocly***

![Redoc.PNG](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/Redoc.png)

- 장점
    
    navigation, body, sidebar 3분할로 구성된 깔끔한 UI
    
    다양한 종류의 문서 템플릿을 보유
    
- 단점
    
    무료형 버젼의 경우, 지원 기능 부족
    

e. **OpenAPI Generator***

![openAPI.png](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/openAPI.png)

- 장점
    
    Postman collection 자동지원
    
    약 40여 개의 프로그래밍 언어로 Server stubs 생성 가능 
    
- 단점
    
    spec 작성 시 오류가 발생하면 Generator로 생성된 코드에도 오류가 발생
    
    유저풀이 좁아 솔루션을 얻기 어려움
    

f. **Swagger** 

![swagger.png](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/swagger.png)

- 장점
    
    API Call TEST 기능이 있어서 개발 소요 시간을 대폭 줄일 수 있음
    
    유저풀이 넓어 솔루션을 얻기 쉬움
    
    AWS 람다와 AWS API gateway에 배포가능
    
- 단점
    
    직관적이지 않은 UI
    
    cloud microservice 호환성 문제
    

## 결론

<aside>
💡  UI나 전체적인 디자인이 부족하지만, 개발 소요시간을 줄일 수 있고 유저풀이 넓어
솔루션을 쉽게 얻을 수 있는 **Swagger**가 가장 적합할 것으로 생각

</aside>

## 번외: Express 서버 내 Router에 OpenAPI Spec 정의

→  swagger-node-express 미들웨어 활용 

```jsx
var findById= {
  'spec': {
    "description": "Operations about pets",
    "path": "/pet.{format}/{petId}",
    "notes": "Returns a pet based on ID",
    "summary": "Find pet by ID",
    "method": "GET",
    "parameters": [swagger.pathParam("petId", "ID of pet that needs to be fetched", "string")],
    "type": "Pet",
    "errorResponses": [swagger.errors.invalid('id'), swagger.errors.notFound('pet')],
    "nickname": "getPetById"
  },
  'action': function (req,res) {
if (!req.params.petId) {
throw swagger.errors.invalid('id');
    }
    var id= parseInt(req.params.petId);
    var pet= petData.getPetById(id);

if (pet) {
      res.send(JSON.stringify(pet));
    }else {
throw swagger.errors.notFound('pet');
    }
  }
};

swagger.addGet(findById);

```

- 장점
    
    API 변경 시, 바로 spec에 반영 가능
    
- 단점
    
    spec 적용을 위해 기존과 다른 방식으로  express  코드 작성 필요
    
    해당 미들웨어 지원이 종료된 것으로 보임 
    

### OpenAPI docs 내 Error state 개선 방안

 

![Untitled](API%20docs%20Tool%20%E1%84%8B%E1%85%A6%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%E1%86%AF%20ac7c37a91e8e4a0887aa43cf244fe18c/Untitled%201.png)

 각 엔드포인트에 대해 HTTP 상태 코드마다 한 개의 응답만이 가능한 상황. 즉, 한 개의 HTTP 상태 코드 에 한 개의 schema만 적용 가능. 
 따라서 같은 코드에서 여러 개의 답이 필요한 경우, 페이 로드에 메세지랑 코드를 보내어 하단에 따로 작성된 custom 코드에서 대조하는 방식으로 구현되었음. 

→ OpenAPI 2.0 (a.k.a swagger2.0) 에서는 정규 HTTP 상태 코드만 지원하며, 개발사에서도 custom 코드와 메세지는 에러 응답 내 바디로 제공하라는 입장

> **How to have a custom status code in response**
**링크:** [https://community.smartbear.com/t5/Swagger-Open-Source-Tools/How-to-have-a-custom-status-code-in-response/td-p/208904](https://community.smartbear.com/t5/Swagger-Open-Source-Tools/How-to-have-a-custom-status-code-in-response/td-p/208904)
> 

 OpenAPI 3.0 에서 oneOf, anyOf 속성을 이용하여, 다수의 응답을 넣는 기능을 지원한다고 되어 있으나, 버그 때문인지 렌더링이 되지 않아 현재 답보 상태임

```yaml
openapi: 3.0.0
...
paths:
  /path:
    get:
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/ResponseOne'
                  - $ref: '#/components/schemas/ResponseTwo'
```

> **Multiple responses using oneOf attribute do not appear in UI**
**링크:** [https://github.com/swagger-api/swagger-ui/issues/3803](https://github.com/swagger-api/swagger-ui/issues/3803)
> 

 

 따라서 OpenAPI 2.0 에서 구현된 현재 방법이 가장 최선으로 생각되며, 향후 3.0으로 migration 시, 상기 방법으로 다수의 schema 구현 여부 확인 필요. 

> **Top 10 API Documentation Tools**
**링크:** [https://blog.api.rakuten.net/api-documentation-tools/](https://blog.api.rakuten.net/api-documentation-tools/)
> 

> **5 Best API Documentation Tools**
**링크:** [https://blog.dreamfactory.com/5-best-api-documentation-tools/](https://blog.dreamfactory.com/5-best-api-documentation-tools/)
> 

> **[B2] OpenAPI Specification으로 타입-세이프하게 API 개발하기: 희망편 VS 절망편**
**링크:** [https://www.youtube.com/watch?v=J4JHLESAiFk](https://www.youtube.com/watch?v=J4JHLESAiFk)
>
