---
layout: post
title: 'VS Code Custom Snippets 추가' 
author: teamsmiley 
date: 2016-08-15
tags: [program]
image: /files/covers/blog.jpg
category: {program}
---
# VS Code Snippets Mark Down 추가

VS Code 로 마크다운을 작성하다 보면 중간에 딱 걸리는 부분이 있다 

image 를 넣으려면 어떻게 하지? 

macosx command key html 코드는 뭐지?

이런 부분을 snippets로 해결이 가능하다고 해서 해보려 한다. 

일단 Code >> Preference >> User Snippets 를 실행한다. 

![]({{site_baseurl}}/assets/vs-code-snippets01.png)

어떤 언어를 쓸건지 물어본다.  나는 Markdown 을 선택한다. 

![]({{site_baseurl}}/assets/vs-code-snippets02.png)

markdown.json 파일이 열린다. 

![]({{site_baseurl}}/assets/vs-code-snippets03.png)

다음 코드를 추가한다. 

```
	"github_image": {
		"prefix": "imageurl",
		"body": [
			"![]({{site_baseurl}}/assets/$1)"	
		],
		"description": "smiley_markdown"
	},
	"command_key": {
		"prefix": "command_key",
		"body": [
			"&#8984;"
		],
		"description": "mac_command_key"
	}
```

간단하게 설명하면 prefix는 내가 무엇인가를 입력햇을때 인텔리센스에 나오게 하는것이다. 

imageurl을 치면 인텔리센스가 나오고 거기에서 탭을 치면 body에 넣은것들이 자동으로 실행되는 것이다. 

command_key는  맥에 command  key 인데 매번 코드르 외워서 입력할 수 없어서 위와 같이 해뒀다.

이제 VS Code를 재시작하고 imageurl 까지만 치면 인텔리센스가 나오는걸 알수가 있다. 
거기에서 탭을 치면된다. 

![]({{site_baseurl}}/assets/vs-code-snippets04.png)

자세히 보면 코드 snippet가 아닌것도 나온다. 잘 선택해서 탭을 해야한다. 





