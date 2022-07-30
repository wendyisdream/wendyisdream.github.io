---
title: github blog
author: Wendy
categories: [Blogging, 개발환경]
tags: [개발환경]
---

### GitHub 에 블로그 만들기 - jekyll 이용

개발 블로그 플랫폼을 찾아보다가, GitHub 를 이용할수 있다는 것을 알게 되어 적용해 보았다. 바로 이곳!:)  
GitHub 블로그는 jekyll 을 사용하여 사이트를 생성한다고 한다. 

Jekyll 이란?  
<https://en.wikipedia.org/wiki/Jekyll_(software)>  

jekyll을 로컬 개발 서버에서 실행하여 확인해볼때 사용법은 공식 사이트와 테마의 가이드 문서가 가장 정확했다. (영어 독해가 바로 안됬을 뿐...)

#### jekyll themes 

jekyll 테마를 제공하는 여러 사이트가 있다.  
맘에 드는 테마를 골라 사용!

- <https://jamstackthemes.dev>
- <https://jekyllthemes.io>
- <https://jekyll-themes.com>

#### 선택한 테마를 내 repository로 복사하기.

github 블로그는 github repository로 관리된다.
테마의 github에서 fork해서 가져오는 방법과   
파일을 복사해서 가져오는 방법이 있다.  
테마의 공식 repository를 수정할게 아니라면 복사하는 방법이 더 좋을 것 같다.


#### 로컬 확인을 위한 환경 갖추기

서버에 올리기전 로컬에 jekyll 환경을 갖춰서 확인해볼 수 있다.

- 설치 하자.
```console
sudo apt install ruby-bundler
sudo apt install ruby-full build-essential zlib1g-dev
sudo apt install bundler
sudo gem install jekyll
```

- blog 소스가 위치한 곳에서 실행 (1회 실행)

```console
bundle install
bundle add jekyll
```

- 서비스 실행

```console
bundle exec jekyll s
```


#### 로컬에서 확인

다음의 주소로 확인해 볼 수 있다.

http://127.0.0.1:4000


#### 설정, style등 나의 정보로 업데이트하기

기본 설정 정보는 _config.yml 에 있다. 각 테마에서 제공하는 가이드에 따라 설정을 진행하면 된다.
GitHub 블로그의 페이지는 Github의 다른 문서와 마찬가지로 markdown 언어로 작성하면 된다.  

