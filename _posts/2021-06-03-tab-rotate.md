---
layout: post
title: 'chrome tab rotate extension'
author: teamsmiley
date: 2021-06-05
tags: [devops]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# 탭을 로테이션 시켜주는 크롬 익스텐션

주식차트를 계속 로테이션으로 보여주는 생각을 하다가 찾아보았다.

## github repository

설정을 저장할 git repo를 하나 만들었다.

다음 파일을 만들고 커밋/푸시한다.

```json
{
  "settingsReloadIntervalMinutes": 1,
  "fullscreen": false,
  "autoStart": false,
  "lazyLoadTabs": false,
  "websites": [
    {
      "url": "https://elite.finviz.com/quote.ashx?t=NET",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=AMZN",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=TSLA",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=SOXL",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=NVDA",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=MRNA",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=TQQQ",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=FNGU",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=SNOW",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=ABNB",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    },
    {
      "url": "https://elite.finviz.com/quote.ashx?t=AAPL",
      "duration": 10,
      "tabReloadIntervalSeconds": 5
    }
  ]
}
```

열고싶은 주소들은 전부 적고 duration, reload 를 적어주면 된다.

저장하고 raw url을 복사해둔다.

![]({{ site_baseurl }}/assets/2021-06-03-tab-rotate/2021-06-05-17-16-09.png)

<https://raw.githubusercontent.com/teamsmiley/tab-rotate/main/tab-rotate.json>

## tab rotate extension 설치

<https://chrome.google.com/webstore/detail/tab-rotate/pjgjpabbgnnoohijnillgbckikfkbjed?hl=en-GB>

설치후 설정에 위에서 복사해둔 설정파일 주소를 넣는다.

<https://raw.githubusercontent.com/teamsmiley/tab-rotate/main/tab-rotate.json>

## extension 활성화

![]({{ site_baseurl }}/assets/2021-06-03-tab-rotate/2021-06-05-17-17-57.png)

## 확인

이제 새창을 열고 익스텐션을 클릭한다.

![]({{ site_baseurl }}/assets/2021-06-03-tab-rotate/2021-06-05-17-19-42.png)

새창이 열리면서 10초마다 탭이 바뀌는것을 볼수있다.

이제 한쪽 구석에 있던 컴퓨터에 모니터를 붙여두고 하루종일 반복되게 해두면 주식의 변화를 볼수가 있다.
