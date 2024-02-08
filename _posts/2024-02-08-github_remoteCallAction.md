---
layout: post
title:  "Github actions를 원격으로 호출해보자"
date:  2024-02-08 01:00:00 +0900
categories: github
tags: github cheatsheet
---

## Summary

Github repository의 외부에서 action을 호출하고 싶을 때가 있다. 이를 위해 github actions의 trigger 중에는 `repository_dispatch`라는 webhook을 처리하는 event trigger가 있다.  
이를 사용해서 외부에서 actions를 호출하여 실행시켜보도록 하자.

### github 호출하기

TAG 정보를 `remote_actions/${date}.txt` 파일에 저장하는 action을 만든다. jobs에 들어가는 shell script는 아래의 과정을 따르도록 하였다.  
git actions에서 호출되는 이벤트는 `types`이며, 사용하는 데이터는 `payload`에 담긴다.

- github checkout/commit을 위해 github token을 주입받아야 한다.
- git 설정정보 설정. 여기서는 github에서 제공하는 bot을 사용한다.
- tag 정보가 없으면 error를 발생시킨다.
- payload에 담겨온 tag 정보를 지정된 경로(`remote_actions/${date}.txt`)에 작성한다.
- git에 해당 파일을 commit/push 한다.

```yaml
name: Remote call and write content to file

on:
  repository_dispatch:
    types: [ remote_call ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Update version
        run: |
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git config --global user.name "github-actions[bot]"
          if [ -z "$TAG" ]; then
            echo "No message found"
            exit 1
          fi
          mkdir -p ./remote_actions
          export FILENAME=./remote_actions/$(date +'%Y-%m-%d_%H:%M:%S').txt
          export MESSAGE=${{ github.event.client_payload.message }}
          echo $MESSAGE > $FILENAME
          git add $FILENAME
          git commit -m "Remote called, saved to $FILENAME"
          git push
```

### 만들어진 actions를 호출하기

github에서는 webhook을 위한 RESTApi를 지원하고 있으며 당연하지만 이 내용은 curl을 통해 호출이 가능하다.
하지만 아무나 이 webhook을 호출할 수는 없고, 권한이 있는 사용자만이 이를 호출할 수 있다. 여기서는 github의 PAT token을 사용하여 인증을 처리하도록 하였다. 그리고 이 PAT 토큰은 본인의 github의 `developer settings`에서 만들 수 있다. 만드는 방법은 [Github: PAT token 생성 방법](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#about-personal-access-tokens) 를 참고하자.

#### 선행 조건

- PAT 토큰: github의 설정에서 외부 접속을 위해 만드는 토큰이다. repository 권한은 반드시 필요하다.
- X-Github-Api-Version: github에서 릴리즈한 api 버전을 의미한다. 해당 버전으로 api를 호출하게 되며, 현재 릴리즈는 `2022-11-28`이다.

#### Github 이벤트 실행

위 github actions에서 작성한 이벤트는 `remote_call`이며 payload에는 `tag`가 필요하다. 해당 내용을 body의 `event_type`과 `client_payload`에 넣어 API를 호출한다.

```terminal
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ${여기에 본인의 github PAT를 넣어줍니다}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/soongwonjun/action-test/dispatches \
  -d '{"event_type":"remote_call","client_payload":{"tag": "1.0.1"}}'  
```

  ## Reference
  - [Github: API Version](https://docs.github.com/ko/rest/about-the-rest-api/api-versions?apiVersion=2022-11-28)
  - [Github: repository_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch)
  - [Github: create dispatch-event](https://docs.github.com/en/free-pro-team@latest/rest/repos/repos?apiVersion=2022-11-28#create-a-repository-dispatch-event)
  - [Github: PAT token 생성 방법](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#about-personal-access-tokens)
