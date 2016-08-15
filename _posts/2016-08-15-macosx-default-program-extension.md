---
layout: post
title: 'macosx에서 파일 확장자에 따른 기본 프로그램 변경' 
author: teamsmiley 
date: 2016-08-15
tags: [program]
image: /files/covers/blog.jpg
category: {program}
---

# macosx에서 파일 확장자에 따른 기본 프로그램 변경 

맥을 자주 사용해보려고 시도한지 2주 하나의 불편한점 이 발견됬다. 
메모를 위해서 마크다운 파일을 작성하는데 마우스로 더블 클릭하면 항상 xcode가 실행된다. 

markdown을 위해서 vs code를 사용하기로 정햇으므로 매번 이걸 오른쪽 버튼을 눌러서 선택을 해줘야한다...
상당히 불편한 상태이다. 

![]({{site_baseurl}}/assets/default_application01.png)


수정해보자. 

1. 원하는 파일확장자를 가진 파일을 선택후 오른쪽 버튼을 누르고 GetInfo를 선택한다. 

1. open with가 보일것이다. 이프로그램이 계속 실행이 된다. 

![]({{site_baseurl}}/assets/default_application02.png)

1. 프로그램을 수정해보자. 그러면 change all이 활성화 된다. 

![]({{site_baseurl}}/assets/default_application03.png)

1. 다시 확인하는 창이 뜨고  continue를 한다. 

![]({{site_baseurl}}/assets/default_application05.png)

1. 이제 완료 되었다.  다시 원하는 파일을 클릭후 오른쪽 버튼을 눌러보자. 

![]({{site_baseurl}}/assets/default_application07.png)

변경되 있는것을 확인할수 있다. 

이제 아이콘을 더블클릭을 하면 내가 원했던 프로그램이 실행 되는것을 볼수가 있다.
