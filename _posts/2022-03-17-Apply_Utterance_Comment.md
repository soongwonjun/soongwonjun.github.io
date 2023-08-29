---
layout: post
title:  "Jekyll에 comment를 적용하기"
date:  2022-03-17 00:30:00 +0900
categories: github
tags: jekyll github
description: Github에 utterance comment를 적용하고, theme를 prefers-color-schema에 따라 다이나믹하게 바꾸어보자
---
## 사용된 스텍

- [Jekyll](https://jekyllrb-ko.github.io)
- [Utteraces](https://utteranc.es/)

## 선택한 이유

- Git repository의 issue에 title별로 볼 수 있다.
- disqus보다 빠르다.
- 적용이 쉽다.
- 별도의 repository 생성 필요 없이 blog로 사용중인 repository에 바로 연결할 수 있다.

## 단점

- theme가 제한적이다. 보안적 이유로 인하여 custom theme는 지원하지 않는다.

## 코멘트 적용하기

- public으로 열린 Git repository를 먼저 만들어야 하지만, 여기서는 이미 블로그로 사용중인 repository를 사용한다.  
  코멘트를 다른 페이지가 아닌, blog repository에서 바로 보고 싶기 때문이다.
- [Utteraces](https://utteranc.es/) 페이지에서 필요한 정보를 입력하면, 페이지 하단의 dom script에 바로 정보가 채워지게 된다.
  - Repo 주소, 매핑 방법(여기서는 title로 매핑하였다), label을 입력하면 바로 utterances dom이 출력된다.
- 해당 페이지를 blog에 붙여넣으면 종료된다.
  - 여기서는 post.html에 붙여넣었다.

```html
<script src="https://utteranc.es/client.js"
      repo="soongwonjun/soongwonjun.github.io"
      issue-term="pathname"
      label="blog-comment"
      theme="github-light"
      crossorigin="anonymous"
      async>
</script>
```

## 스크립트로 테마가 동적으로 바뀌게 하기

이 블로그는 light-mode와 dark-mode를 지원하는 테마를 사용중이다.  
때문에 테마도 지원되어야 이쁘다.  
하지만 utterances는 custom-theme를 지원하지 않기 때문에 스크립트를 사용해서 구현한다.

```javascript
// light-mode/dark-mode는 prefers-color-scheme로 구분된다.
// 여기에 listener를 걸어서 변화를 확인하고, utterances iframe에 post를 보내어 theme의 동적인 변화를 가한다.
// 다만 prefers-color-schema가 바뀌고, iframe에 새 theme가 적용될 때 까지는 딜레이가 생긴다.
window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
  let message = {
    type: 'set-theme',
    theme: event.matches ? 'github-dark' : 'github-light'
  };
  const iframe = document.getElementsByClassName('utterances-frame')[0];
  iframe.contentWindow.postMessage(message, 'https://utteranc.es');
});
```
  
다른 방법이라면, light-mode/dark-mode dom을 2개 만들어서 display속성을 바꾸어 표현하는 방법이 있다.  
이 때는 prefers-color-schema가 바뀔 때 dom이 다시 그려지기까지 시간이 매우 짧다.  
다만 이 방법은 한쪽의 comment 변경 내용이 다른쪽 dom에 반영되지 않는다.  
utterances의 comment는 처음 dom이 만들어질 때, 그리고 내가 comment를 남길 때에만 바뀌기 때문이다.

```html
<div class="utterance-comment-area">
   <div id="utterance-light-mode-comment"> ... </div>
   <div id="utterance-dark-mode-comment"> ... </div>
</div>

<script>
  let domLight = document.getElementById('utterance-light-mode-comment');
  let domDark = document.getElementById('utterance-dark-mode-comment');
  window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
    let target = event.matches ? domDark : domLight;
    if (event.matches) {
      domLight.style.display = 'none';
      domDark.style.display = 'block';
    } else {
      domLight.style.display = 'block';
      domDark.style.display = 'none';
    }
  })
</script>
```

이 블로그에서는, 코멘트 반영 내용이 계속 유지되길 바라는 마음에 첫 번째 소개한 방식대로 구현하였다.

## Appendix & References

- [utterance: issue/549](https://github.com/utterance/utterances/issues/549)
