---
layout: post
title: "Git pull request / merge 이슈 대응기"
excerpt: ""
subtitle: "Git"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-02-15
tags: [Git]
---

# Git pull request / merge 이슈 대응기

aws: Git
: 박상우

서비스 중인 Prod Repository에 개발 중인 코드가 섞이는 이슈가 생겼을 때 어떻게 대응하였는지를 기록합니다.

![Untitled](https://1drv.ms/i/c/cdf0612ba3befd25/IQMLdZ8hRRiVTp2eaUqMkzO5AXP_qitWDe8rOONHxdhbUjM?width=660)

서비스 중인 레포지토리(이하 havah)에서 1.3 버전 대응을 위해 fork로 신규 레포지토리(이하 havah 1.3)를 새로 생성하여 작업 중인 상황이었습니다. 

- havah - prod가 돌아가는 main repository를 칭함
- havah1.3 - 1.3 버전 용 sub repository를 칭함

havah1.3 내 main 브랜치에서 test env로 배포를 진행하려 local에서 커밋/푸시 후 github에서 local → main으로 merge를 진행했으나... 

그러나 main branch는 havah1.3이 아닌 havah 였고, 결국 메인 브랜치에 local 작업분이 섞이는 이슈 발생하고 말았습니다 :(

![스크린샷 2022-11-10 오후 3.32.42.png](https://1drv.ms/i/c/cdf0612ba3befd25/IQPQjRa1KPOYSLLccEM6sNdVARnUST31XnE1XW5Tp-7Hfsc?width=660)

요약하자면, 

1. havah1.3 - referralCheck 브랜치(local)에서 작업 후 커밋
2. havah1.3 - main branch에 merge / pull-request를 하려 했으나 havah - main으로 merge 진행됨
3. 즉, 현재 서비스 되고 있는 havah - main에 1.3 작업분이 섞임

이에 다음과 같이 조치했습니다~

1. local에 있는 havah - main으로 git pull을 해옴 -> 버전 기록과 섞인 코드 받아옴
2. 10/21일자 광현님이 마지막으로 올린 git으로 roll-back 및 main2로 새롭게 브랜치를 따서 백업
3. main3에 추가 백업
4. main에서 10/21일자 버전으로 revert (**hard option**) 진행
5. havah-main 으로 commit / push 진행
6. 추가 commit/push를 통해 remote와 local 간 연결 여부, 차이없는지 체크
7. 다른 작업자의 local branch로 pull을 받아와 업데이트 내용 있는지 더블 체크

revert는 서비스 시 가능하면 사용을 지양해야 하고 동일한 이슈 발생 시, 다음과 같은 방법으로 대응할 예정입니다. 

1. 기존 local havah-main 브랜치 이름을 변경
2. remote에서 havah-main을 pull 받은 다음, 이름 변경
3. 1에서 바꾼 브랜치 이름을 main으로 바꾼 다음 커밋/푸쉬

<aside>
💡 pull request / merge 사용을 지양하되, 어쩔 수없이 하게 된다면 base repository 확인!!!
</aside>
