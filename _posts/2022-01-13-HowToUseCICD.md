---
layout: post
title: "자동화 배포 사용법"
excerpt: ""
subtitle: "CI/CD"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-11-16
tags: [CI/CD]
---

# 자동화 배포 사용법

Github action을 활용하여 CI/CD를 해봅시다. 

## 📘 Introduction

- 배포는 Git action 사용 중이며, **새로운 릴리즈 작성 및 퍼블리싱 시**, 배포 자동화가 작동됨
릴리즈 관련 탭은 Github 레포 메뉴에서 우측 중간에 위치

- 현재 ‘dev’ 브랜치의 소스 코드가 배포되고 있으며, 어떤 브랜치의 소스코드가 배포될 지는 
git repo 내 [.github/workflows](https://github.com/2bytes-platform/tb2p-web-react/tree/main/.github/workflows)/[s3-deploy-dev-2p.yml](https://github.com/2bytes-platform/tb2p-web-react/blob/main/.github/workflows/s3-deploy-dev-2p.yml) 에서 설정 가능

- 릴리즈 작성 시 하기 스크린샷과 같이 새로운 태그(create new tag)를 생성해야 자동화 trigger가 발동됨.

![Untitled](%E1%84%8C%E1%85%A1%E1%84%83%E1%85%A9%E1%86%BC%E1%84%92%E1%85%AA%20%E1%84%87%E1%85%A2%E1%84%91%E1%85%A9%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%87%E1%85%A5%E1%86%B8%207454e5b9f7eb4793bc18eef2257bbe51/Untitled.png)

- 기획팀이 작성한 monday - 릴리즈 노트 내의 버저닝을 릴리즈 작성 시, 기재 필요 
(현재는 버저닝 동기화 안되어 있음)

- 배포 과정은 액션 창에서 볼 수 있으며, 해당 배포 클릭 시, 로그 확인 가능.

![Untitled](%E1%84%8C%E1%85%A1%E1%84%83%E1%85%A9%E1%86%BC%E1%84%92%E1%85%AA%20%E1%84%87%E1%85%A2%E1%84%91%E1%85%A9%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%87%E1%85%A5%E1%86%B8%207454e5b9f7eb4793bc18eef2257bbe51/Untitled%201.png)

> GitHub action 과금은 배포 시간으로 계산되며, 한달에 2000분을 무료로 사용 중
> 

- 배포가 끝나면, AWS S3에 클라이언트 정적파일이 업로드 된 것을 확인할 수 있으며, 
AWS 계정은 슬랙채널로 공유함

- CDN 배포 전 s3에 업로드 된 정적 클라이언트 파일 확인 가능
 [http://boracat.io.s3-website.ap-northeast-2.amazonaws.com](http://boracat.io.s3-website.ap-northeast-2.amazonaws.com/)

- CDN 배포는 AWS CloudFront에서 진행하며, [boracat.io](http://boracat.io/) - Create invalidation 클릭

![Untitled](%E1%84%8C%E1%85%A1%E1%84%83%E1%85%A9%E1%86%BC%E1%84%92%E1%85%AA%20%E1%84%87%E1%85%A2%E1%84%91%E1%85%A9%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%87%E1%85%A5%E1%86%B8%207454e5b9f7eb4793bc18eef2257bbe51/Untitled%202.png)

- **/*** 를 입력하면 CDN 서버 내에 있는 모든 디렉토리에 있는 파일을 s3에서 가져온 데이터로 교체하는 작업이 진행됨 (약 10초 내외)

- 무효화 작업 이후, 캐시 날리기 혹은 incognito(프라이빗) 모드로 [https://boracat.io](https://boracat.io/) 로 접속하여 
배포가 잘 되었는지 확인

## 📋 참조

[](https://www.notion.so/2bytes/GitHub-Actions-CI-CD-1277b3cf8dfa455cad224c7e65d856c0)
