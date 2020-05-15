---
layout: post
title: "Jekyll에 emoji 추가하기"
excerpt: "emoji를 사용해 포스팅 하기"
categories: [jekyll]
comments: true
tags:
  - Jekyll
---

*jemoji 플러그인을 설치해 포스트에 귀여운 이모티콘을 넣어보자!!!!*

# 설치 방법
`Gemfile`에 아래 추가
```
gem 'jemoji'
````
추가 후, 다음 명령을 실행한다. 
```
bundle install
```

`_config.yml`에 아래 추가
```
plugins:
  - jemoji
```
마찬가지로 다음 명령을 실행해 플러그인을 설치한다. 
```
gem install jemoji
```



# 사용

포스트 내에서 `:heardpulse:` 이런 식으로 사용할 수 있다.

더 많은 이모티콘은 https://www.webfx.com/tools/emoji-cheat-sheet/ 여기서 찾을 수 있다. 

---

나는 설치 과정에서는 문제가 없었지만,
알 수 없는 이유로 이모지 변환이 되지 않는다...ㅠㅠ

그래도 나중에 다시 보고 해보려고 글은 쓴다!!!!!!!
나중에 깃허브 이슈에 올려봐야겠다. 

# 참조
jemoji repo: <https://github.com/jekyll/jemoji>
