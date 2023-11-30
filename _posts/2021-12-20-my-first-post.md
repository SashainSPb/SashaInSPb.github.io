---
layout: post
title: "📅 github pages를 이용한 정적 웹사이트 구성하기"
excerpt: "github pages 이용하여 웹사이트 구성해보자"
subtitle: ""
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2021-12-20
tags: [github_pages]
---

## guthub blog를 생성하는 과정에 대해서 서술하겠음.  
해당 블로그를 참조하여 github pages를 구성하였음  
[GitHub Pages 블로그 소개](https://devinlife.com/howto%20github%20pages/github-blog-intro/)

Jekyll은 루비로 만든 정적 웹사이트 생성기이며, md로 작성된 컨텐츠를 html로 변환해주는 역할을 갖고 있음.

다음과 같은 순으로 포스팅을 시작하자

1. md로 컨텐츠를 생성한다  
2. 로컬에서 bundle exec jekyll serve를 실행하여, 검수  
3. 검수 후, 연결된 remote repo로 commit & push 하면 끝  


## 다음과 같은 순으로 환경을 구축하자

- ruby 설치  
- Jekyll과 bundler 설치  
  - 최신버전으로 생성필요  
  - 'jekyll new HelloBlog'로 샘플 블로그를 생성
  - 'bundle exec jekyll serve'
- minimal-mistakes theme 세팅, [<mmistakes repo></mmistakes>](https://github.com/mmistakes/minimal-mistakes)에서 로컬로 clone 후, 필요없는 파일은 삭제  
  bundle 명령어를 실행하여 필요한 라이브러리를 다 설치해주자 [<mmistakes guide></mmistakes>](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)  
  gem minimal-mistakes-jekyll로 설치 
- username.github.io로 repo 생성 후, public 설정
- remote repo와 local repo를 연결하자

- [`require': cannot load such file -- webrick (LoadError)](https://junho85.pe.kr/1850)
- git 설치해서 bash 커맨드창 이용 
- 검색 기능 지원하는지 찾아보자 
- minimal mistakes Gemfile 관련 오류 발생 시, 직접 복붙해서 붙여넣자
  ```ruby
  source "https://rubygems.org"

  # Hello! This is where you manage which Jekyll version is used to run.
  # When you want to use a different version, change it below, save the
  # file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
  #
  #     bundle exec jekyll serve
  #
  # This will help ensure the proper Jekyll version is running.
  # Happy Jekylling!

  # gem "github-pages", group: :jekyll_plugins

  # To upgrade, run `bundle update`.

  gem "jekyll"
  gem "minimal-mistakes-jekyll"

  # The following plugins are automatically loaded by the theme-gem:
  #   gem "jekyll-paginate"
  #   gem "jekyll-sitemap"
  #   gem "jekyll-gist"
  #   gem "jekyll-feed"
  #   gem "jekyll-include-cache"
  #
  # If you have any other plugins, put them here!
  # Cf. https://jekyllrb.com/docs/plugins/installation/
  group :jekyll_plugins do
  end
  gem "webrick", "~> 1.7"
  ```
trouble shooting: [mmistakes Installation](https://mmistakes.github.io/minimal-mistakes/docs/installation/)