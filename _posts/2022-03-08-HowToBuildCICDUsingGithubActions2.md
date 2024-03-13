---
layout: post
title: "GitHub actions를 이용한 CI CD 구축하기 - 2"
excerpt: ""
subtitle: "GitHub action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-03-08
tags: [GitHub]
---

# GitHub Actions를 이용한 CI/CD 구축하기 ②

GitHub Actions를 이용한 CI/CD 구축하기 ① 에 이은 두번째 글입니다.

## 📘 Menu

다음과 같은 순으로 배포 자동화 기능을 구현

**6. Slack slash command 기능을 이용한 수동 발동 조건 추가**

**7. 개발 / 운영  서버 각각 GitHub action 발동 조건 추가 (1/10)**

## 📘 Introduction

### **Slack slash command 기능을 이용한 수동 발동 조건 추가**

- Slack 에서 ‘/deploy’ 커맨드를 이용해서 자동화 배포를 구현하는 Cool한 방법이 있어 구현하였음. Slack 에서 제공하는 App과 AWS API Gateway 서비스가 필요
- 기존 workflows를 정의한 yaml 파일 내에 ‘repository_dispatch’ 프로퍼티를 추가. types에는 해당 trigger 명을 기재
    
    ```yaml
    on:
      # Triggers the workflow on push or pull request events but only for the main branch
      repository_dispatch:
        types:
          - deployment
    ```

<aside>
✅ repository_dispatch는 GitHub api를 활용하여 외부에서 Actions를 발동 가능케 하는 옵션

</aside>

- GitHub에서 repository_dispatch API를 이용하기 위해서는 personal access token과 repository owner 정보가 필요. access token 생성은 아래 url에서 가능하며, repo scope가 필요하므로 체크  [https://github.com/settings/tokens](https://github.com/settings/tokens)
    
![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-054241.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQOzcpvI0CxBQpsYrE_p8lPDARYOU_ZzR8dcr1Waaye1KKw?width=660)
    
- 발급받은 token 정보와 repository owner 정보를 넣은 curl을 날리면, GitHub Actions 발동 확인
    
    ```shell
    curl \
      -X POST \
      -H "Accept: application/vnd.github.v3+json" \
      -H "Authorization: Bearer ********* PERSONAL ACCESS TOKEN *********" \
      https://api.github.com/repos/{owner}/actions-experiments/dispatches \
      -d '{"event_type":"deployment"}'
    
    ```
    
- GitHub 서버에서 이해할 수 있는 payload 형태로 보내주는 방식을 Slack에서는 지원해주지 않는 것으로 보이며, 이에 AWS API Gateway 서비스를 도입

<aside>
⚠️ Slack은 **‘application/x-www-form-urlencoded'** 형식의 payload를 보낼 수 있고, Github은 ‘**application/json’** payload를 받음.
</aside>

- 먼저, Create new API 에서 API 유형은 ‘REST API’, API 이름 설정 및 엔드포인트 유형은 ‘지역’으로 설정

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-070310.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQOzcpvI0CxBQpsYrE_p8lPDARYOU_ZzR8dcr1Waaye1KKw?width=660)

- 리소스 path 와 리소스 이름을 넣어 생성
    
![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-070657.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPo2XuXtQ_zT7FNQ667JdTdATeOjSWqtKrM3fuwMafJq7I?width=660)
    
- POST 형식의 메서드를 추가. AWS 이외의 서비스를 호출할 수 있도록 통합 유형은 HTTP로 설정.엔드포인트 url은 [https://api.github.com/repos/2bytes-platform/2bytescorp.com/dispatches](https://api.github.com/repos/2bytes-platform/2bytescorp.com/dispatches)로 입력

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-071140.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPpyKHWnRCaQLKlyOD5LRANAf1W_VCRYfaekU3dMLXSc3g?width=660)

- 통합 요청을 클릭하면 메서드 실행 보기가 표시되는데, 통합 요청 단계로 접속하면 하기와 같은 메뉴가 보임. HTTP 헤더에 Accept, Authorization을 넣고 value로는 각각 ‘application/vnd.github.v3+json'과 'Bearer ${Personal access token}’을 입력

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-071738.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPXa_Jkfb3CTqXOoyqkw0KVAUk8bQKpi5De2b_jl9WYDZ4?width=660)

- 위의 작업을 완료했다면, GitHub으로 API testing이 가능하며, 메서드 실행보기 내 노란색 음영 클릭.이후 요청 본문에 `{"event_type":"deployment"}` 입력 후, test 버튼을 누르면 GitHub Actions가 발동되는지 확인 가능

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-072439.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQP37PHHgrS-SbIS6QCLa6qoAWeIRoN-5MavDIWun80dSQU?width=660)

- Slack은 **‘application/x-www-form-urlencoded'** 형식의 payload를 보낼 수 있고, Github은 ‘**application/json’** payload를 받음. 통합 요청 하단에 매핑 템플릿 메뉴에서 form을 전환해주는 과정이 필요하며 다음과 같이 입력

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-074148.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPEUewLsl17RJEvO0YfxCTsAWAGP7eVAqilZ2RdQTghdVQ?width=582&height=487)

- 마지막 단계로, AWS API 세팅이 끝나면 API 배포를 실행.

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-074521.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPEUewLsl17RJEvO0YfxCTsAWAGP7eVAqilZ2RdQTghdVQ?width=582&height=487)

<aside>
⚠️ 기타 변경이 있을 경우, 변경사항이 자동으로 반영되지 않기 때문에 API **재배포** 필요

</aside>

- 스테이지 이름과 설명을 기재 후 요청 url을 잘 메모해두자. 이는 Slack에서 AWS Gateway로 보내는 엔드포인트가 됨

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-074910.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQMERDsugC9xTb4CHLfPk-sHAcrA9Wr-7Fq6bwahR64gt7U?width=660)

- Slack 에서 slash command를 사용하려면 일단 app이 필요하며 아래 url에서 생성 가능[https://api.Slack.com/apps?new_app=1](https://api.slack.com/apps?new_app=1)

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-075357.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPJrcJg8KkkS7mIQArwBGXPATuRj0Ynl7zOcUer7-WW0B8?height=660)

- Slash Commands를 클릭하면 command를 생성할 수 있으며, Request URL에는 앞 전 AWS stage 설정 시 적어둔 url을 기입

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-075837.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQMogJfM3t3gSbt4Vyi1FwmSAT8DYTL7yCFfgOZxw7Fr1bU?height=660)

- 마지막으로 설정한 Slack slash command를 활용하기 위해서, 해당 앱을 사용하고자 하는 워크스페이스 - 채널에 설치 필요

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-080314.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQNUE5KNOmsxSba5MybP-l9WAb5i-gJ5zohi8JTImaCuZCc?width=640&height=403)

- 정상적으로 설치되었다면, 등록한 command가 자동으로 나오게 됨
    
![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220104-081341.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQNkhpZKwl9zS4SrJ7Wbn91MAYVU6ADXrKGcZ1Hq8VHlyJY?width=660)
    
- Slack APP을 설치한 채널에 /deploy 입력 후, 배포 자동화가 되는지 확인

### **개발 / 운영 서버 각각 GitHub action 발동 조건 추가 (1/10)**

- 운영 서버에 정식 배포 전 개발 서버에서 Q/C가 선행되어야 함. 이에 개발 서버와 운영 서버에 각각 배포가 필요하여 Trigger 조건을 분기하는 방법을 서칭하였음.
- ‘.github/workflows’ 디렉토리 내 워크플로우를 정의한 .yml 파일을 따로 생성. 개발 및 운영 서버에 대한 워크플로우를 따로 정의하여 각각 관리할 수 있음.
- 먼저, 개발 서버 CI/CD를 다음과 같이 구축. 개발은 Slack slash command를 이용한 배포로 진행. 운영 서버와의 구분을 위해 name을 변경
    
    ![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220112-032235.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPtkHzqmgd1Q4JqJ6PailY1ATZowQk6uX6nATtiyl27bFA?height=660)
    
- 그 다음, 운영 서버 CI/CD를 위한 .yml 파일을 따로 생성. 운영 서버는 **release 생성**을 이용한 배포로 구현. 개발 서버 배포 및 Q/C 후, release 생성을 통한 운영 서버 배포가 이뤄지는 플로우를 생성 (현재 운영 서버 구축이 되지 않아 차후 해당 url로 변경 예정)
    
    ![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220112-032701.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQOyv8hMr4TKRKvGTURYTOnJAf67-LLKfb6n4xe1-FtNPIw?height=660)
    
- 다른 방법은 하나의 yml 파일 내에 GitHub 에서 지원하는 표현식을 활용하여 분기 조건을 설정해 주는 것이며, 향후 리팩토링 시 참고 예정
    
    ![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%E2%91%A1%20923f9f79645a4c57b94e1b4bfe7befb4/image-20220112-050032.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQM9iVUx3I_YRIDZ4Mr3bUxOAUYIfjQ2wWfRPWCqJHfw-tw?width=566&height=443)
    

### 보완해야 할 기능

- GitHub에 repository_dispatch API를 보낼 때, 보안성 강화 필요
- Slack slash command 사용 시, 이용자 정보가 Slack과 GitHub Actions에 나올 것> 사용자의 GitHub token 정보를 http request에 동적으로 넣을 수 있다면 요구 사항을 모두 충족 가능할 것으로 보이며, 지속 방법 강구
- workflow yaml 파일 리팩토링

## 📋 **참조**

[https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#release](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#release)

[https://geoffrey.dev/blog/triggering-a-github-actions-workflow-from-slack/](https://geoffrey.dev/blog/triggering-a-github-actions-workflow-from-slack/)

[https://stackoverflow.com/questions/57115520/can-i-have-multiple-github-actions-workflow-files](https://stackoverflow.com/questions/57115520/can-i-have-multiple-github-actions-workflow-files)
