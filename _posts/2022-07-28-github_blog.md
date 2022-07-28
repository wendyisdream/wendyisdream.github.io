---
title: github blog
author: Wendy
categories: [Blogging, 개발환경]
tags: [개발환경]
---

### GitHub 에 블로그 만들기 - jekyll 이용

개발 블로그 플랫폼을 찾아보다가, GitHub 를 이용할수 있다는 것을 알게 되어 적용해 보았다. 바로 이곳!:)    
GitHub 블로그의 페이지는 Github의 다른 문서와 동일하게 markdown 언어로 작성하게 된다.
Git repository에서 블로그도 소스처럼 관리하는데, 로컬에서도 먼저 확인해보 올릴 수 있다~
jekyll 테마를 제공하는 사이트에서 맘에 드는 테마를 골라 사용! 

Jekyll 이란?
<https://en.wikipedia.org/wiki/Jekyll_(software)>  

사용법은 공식 사이트와 테마의 가이드 문서가 가장 정확했다. (영어 독해가 바로 안됬을 뿐...)

#### jekyll themes 

- <https://jamstackthemes.dev>
- <https://jekyllthemes.io>
- <https://jekyll-themes.com>

에서 다양한 테마르 제공한다. 이중에 맘에 드는 것 하나 선택!


#### 선택한 테마를 내 repository로 복사하기.

테마의 github에서 fork해서 가져오는 방법과   
파일을 복사해서 가져오는 방법이 있다.  
테마의 공식 repository를 수정할게 아니라면 복사하는 방법이 더 깔끔할 것 같다.


#### 로컬에서의 확인을 위한 환경 갖추기

다음을 설치하고 실행하였다.

- 환경 갖추기
``` console
sudo apt install ruby-bundler
sudo apt install ruby-full build-essential zlib1g-dev
sudo gem install jekyll


< blog 소스가 위치한 곳에서 실행.>
bundle install
bundle add jekyll
```

- 로컬 실행
``` console
bundle exec jekyll s

```


#### 로컬에서 확인

http://127.0.0.1:4000


#### 설정, style등 나의 정보로 업데이트하기

