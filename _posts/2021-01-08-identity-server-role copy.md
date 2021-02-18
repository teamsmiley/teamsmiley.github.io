---
layout: post
title: "macos 업데이트후 dotnet core ssl관련 에러날때"
author: teamsmiley
date: 2021-02-18
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# macos를 big sur로 업데이트후 dotnet core가 안됨.

macos를 업데이트 한후 dotnet core 가 실행이 잘 안된다.

확인결과 ssl관련 이슈 같아보여서 다음처럼 했다.

```bash
dotnet dev-certs https --clean
dotnet dev-certs https
sudo dotnet dev-certs https --trust
```

https://localhost:5001/api/values 확인해보니 잘 된다.
