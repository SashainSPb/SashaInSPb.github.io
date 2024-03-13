---
layout: post
title: "GitHub actions를 이용한 CI CD 구축하기 - 1"
excerpt: ""
subtitle: "GitHub action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-02-15
tags: [GitHub]
---

# GitHub Actions를 이용한 CI/CD 구축하기

GitHub Actions와 AWS, Slack 서비스를 활용하여 CI/CD를 어떻게 구축했는지 기재한 문서입니다.

## 📘 Menu

다음과 같은 순으로 배포 자동화 기능을 구현했습니다.

1. AWS S3 내 버킷을 생성하고 및 정책을 추가합니다. 그리고 정적 웹사이트 호스팅을 설정합니다.
2. AWS IAM을 통한 사용자를 추가하고 Access key 및 Secret Access Key를 발급합니다.
3. GitHub repository 내 Actions yaml 파일을 설정합니다.
4. Slack 알람 서비스를 연동시킵니다. 
5. GitHub 내 releases 기능을 이용한 수동 발동 조건 설정

## 📘 Introduction

### AWS S3 내 버킷 생성 및 정책 추가, 정적 웹사이트 호스팅 설정

- 데이터를 담을 수 있는 버킷을 생성하고, 리전은 서울 기준으로 ap-northeast-2로 설정
- 퍼블릭 액세스가 가능하도록 설정
- 특정 요청에 대해서만 해당 버킷에 접근할 수 있도록 AWS Policy Generator를 활용하여 정책을 생성한다.정책을 작성하면 하기와 같이 JSON form으로 생성

```json
{
    "Version": "2012-10-17",
    "Id": "Policy1639640906894",
    "Statement": [
        {
            "Sid": "Stmt1639640890223",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::2byteshomepage/*"
        }
    ]
}
```

`"Action": "s3:GetObject"로 설정하고, Resource는 버킷 ARN에 /*를 추가하여 기재`

- 버킷 → 속성 탭에서 정적 웹사이트 호스팅을 설정하고, 인덱스 문서와 오류 문서에 해당하는 html 파일명 기재

### AWS IAM을 통한 사용자 추가 및 Access key 및 Secret Access Key 발급

- IAM 에서 사용자를 추가하고, 권한 설정은 기존 정책 직접 연결 → AmazonS3FullAccess로 선택
- AWS 액세스 유형은 프로그래밍 방식 액세스를 선택하여, Github Action에서 사용할 Access key와 Secret Access Key를 발급

![스크린샷 2022-07-21 오후 12.14.03.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQO0bRh1j-XZS5hiwHqS7yDbAVr35TPY2nNXPNXctb7822Q?width=660)

![스크린샷 2022-07-21 오후 12.20.19.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQP-HVaDW3PwQo2pF7Oxs4RgAaOM4VXD8LIlFxLg0Ts5gzI?width=660)

<aside>
⚠️ Secret Access Key는 생성 후, 재 조회가 불가하니  Access Key와 별도로 메모 요망

</aside>

### GitHub repository 내 Actions 설정

- 배포할 소스가 저장되어 있는 repository 내에 발급받은 Access key와 Secret Access Key를 **각각** 등록.환경 변수 관리 라이브러리와 비슷하며, Name은 workflow 내에서 변수로 쓰임.

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%201277b3cf8dfa455cad224c7e65d856c0/image-20211217-061036.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQOixMwGn9psT4fjWKAcdJdgAfMqp-OEXJVIkgsvY0HNxZM?width=660)

- Actions → New workflow → Manual workflow를 선택하고 다음과 같이 세팅 후 commit

```yaml
# This is a basic workflow to help you get started with Actions

name: deploy-to-s3

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  release:
    types: [published]
  repository_dispatch:
    types:
      - deployment
  # issues:
    # types: [opened]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - name: git clone
        uses: actions/checkout@v2

      - name: npm install
        run: npm install

      - name: build
        run: npm run build

      - name: deploy
        env:
          AWS_ACCESS_KEY_ID: "${{ secrets.AWS_ACCESS_KEY_ID }}"
          AWS_SECRET_ACCESS_KEY: "${{ secrets.AWS_SECRET_ACCESS_KEY }}"
        run: |
          aws s3 cp \
            --recursive \
            --region ap-northeast-2 \
            dist s3://2bytescorp.com
      - name: Slack notification success
        uses: 8398a7/action-slack@v3
        with:
          status: ${{job.status}}
          fields: repo,message,commit,eventName,workflow,job,took
          author_name: ${{ github.actor }}

        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

      - name: Slack notification failure
        uses: 8398a7/action-slack@v3
        with:
          status: ${{job.status}}
          fields: repo,message,commit,eventName,workflow,job,took
          author_name: ${{ github.actor }}

        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

        if: failure()
```

- **on**에는 GitHub Actions가 어떤 조건에서 발동(trigger) ****되는지 기재하는 파트임. 현재 master branch (main) 에서 git push와 PR이 이뤄질 때 실행되는 것으로 설정+ (12/20) action 발동 조건을 **release publish** 으로 변경 + (12/29) 외부 action workflow 발동을 위한 **repository dispatch** 추가
- **jobs**는 여러 Step으로 구성되고, 단일 가상 환경에서 실행. 다른 Job에 의존 관계를 가질 수도 있고, 독립적으로 병렬로 실행
- **steps**는 job 안에서 순차적으로 실행되는 프로세스 단위. 현재 자동 빌드 및 사전에 등록한 2가지 AWS access keys 를 이용하여 S3에 자동 배포까지 설정
- `dist s3://2bytescorp.com.site`에는 s3 버킷명을 기재
- `8398a7/action-slack@v3` 은 git action과 Slack 의 연동을 위해 Slack 측에서 제공하는 tool 이며, 마지막 줄에 if: failure()를 삽입하여 분기 조건을 설정

 **+ (2/28) 특정 브랜치만 CI/CD 시,  checkout 에 다음과 같은 조건 추가**

```yaml
# Checkout a different branch
- uses: actions/checkout@v2
  with:
    ref: 'branch name'
```

### **GitHub 내 releases 기능을 이용한 수동 발동 조건 설정**

- 기존에는 git이 repository에 push 시 자동으로 CI/CD가 이뤄짐. 하지만 git push 빈도수가 잦고, 원할 때에만 배포가 이뤄지면 좋겠다는 의견이 있었음. 이에 GitHub releases publish 시에만 배포가 이뤄질 수 있도록 다음과 같이 기능을 구현하였음.
- action 발동 조건을 아래와 같이 **release published** 으로 변경. release 수정 시 발동하게 하고 싶다면 types에 [edit] 추가
    
    ```yaml
    on:
      # Triggers the workflow on push or pull request events but only for the main branch
      release:
        types: [published]
    ```
    
- Github repository 페이지에서 오른쪽 중간에 위치한 Releases 클릭

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%201277b3cf8dfa455cad224c7e65d856c0/image-20220104-042230.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPfq213YGzQSr1QC1_6kXTUAebHPponN16fimDHHIBfE44?width=660)

- draft a new release 버튼 클릭 후, tag 생성 및 하기 publish release 버튼을 누르면 git action workflow 실행

![GitHub%20Actions%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20CI%20CD%20%E1%84%80%E1%85%AE%E1%84%8E%E1%85%AE%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%201277b3cf8dfa455cad224c7e65d856c0/image-20220104-042525.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQOzf5-Tj696T4xRSMhKKniIAR6MSXAWw1OeiOQgHpCLRXI?width=660)

2부에서 계속

## 📋 **참조**

[](https://zzsza.github.io/development/2020/06/06/github-action/)

[Vue Github Action & S3 배포하기](https://hoon1994.tistory.com/m/64)

[Github Actions 달기 (Slack)](https://gurumee92.github.io/2020/10/github-actions-%EB%8B%AC%EA%B8%B0-slack/)

[Github Action에 대한 소개와 사용법](https://velog.io/@ggong/Github-Action%EC%97%90-%EB%8C%80%ED%95%9C-%EC%86%8C%EA%B0%9C%EC%99%80-%EC%82%AC%EC%9A%A9%EB%B2%95)

[Github Action에 대한 소개와 사용법](https://velog.io/@ggong/Github-Action%EC%97%90-%EB%8C%80%ED%95%9C-%EC%86%8C%EA%B0%9C%EC%99%80-%EC%82%AC%EC%9A%A9%EB%B2%95)

[Events that trigger workflows - GitHub Docs](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#release](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#release))

[Triggering a GitHub Actions Workflow with a Slack Slash Command](https://geoffrey.dev/blog/triggering-a-github-actions-workflow-from-slack/)

[https://github.com/actions/checkout](https://github.com/actions/checkout)

# Subpages

[GitHub Actions를 이용한 CI/CD 구축하기 ②]()
