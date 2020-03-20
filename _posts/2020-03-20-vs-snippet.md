---
layout: post
title: 'Visual Studio Code Snippet' 
author: teamsmiley
date: 2020-03-20
tags: [visual studio]
image: /files/covers/blog.jpg
category: {vs}
---

# Visual Studio Code Snippet

Test작성시 중복되는 많은 코드를 전부 타이핑하다 귀찮아졌다. snippet로 처리해보자.

## my code snippets 위치 파악

vs >> tools >> code snippets manager (Ctrl+K Ctrl+B) 

My code snippets 를 클릭하고 위치를 파악한다. 거기에 파일을 만들어야 인식한다.

위 경로로 이동후 snippet를 만든다. 

* XunitTestThrowException.snippet

확장자는 꼭 snippet으로 만들어야한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet">
  <CodeSnippet Format="1.0.0">
    <Header>
      <Title>Throw Exception</Title>
      <Shortcut>
        testt
      </Shortcut>
    </Header>
    <Snippet>
      <Declarations>
        <Literal>
          <ID>method</ID>
          <Default>Handler</Default>
        </Literal>
        <Literal>
          <ID>scenario</ID>
          <Default>SCENARIO</Default>
        </Literal>
        <Literal>
          <ID>exception</ID>
          <Default>EXCEPTION</Default>
        </Literal>
        <Literal>
          <ID>classname</ID>
          <Default>CLASS_NAME</Default>
        </Literal>
      </Declarations>
      <Code Language="CSharp">
        <![CDATA[
    [Fact]
    public async Task $method$_$scenario$_Throw$exception$Exception()
    {
      //Arrange
      var command = new $classname$
      {
        Name = "Name",
      };

      //Act + Assert
      var sut = new $classname$.$classname$Handler(Context, Mapper, _userServiceMock.Object);
      var error = await Assert.ThrowsAsync<$exception$Exception>(async () => await sut.Handle(command, CancellationToken.None));
    }
        ]]>
      </Code>
    </Snippet>
  </CodeSnippet>
</CodeSnippets>
```

## header
header에서 중요한부분은 shortcut이다  이부분에 값을 넣으면 visual studio에서  이 이름을 누르고 탭을 두번 누르면 내용이 채워진다.

## snippet

code가 중요하다. 원하는 내용의 코드를 넣으면 된다. 

이제 중복되거나 바귀어야하는 부분을 처리하기 위해 $XXX$로 하였다. 그러면 declarations를 넣어줘야한다.

```xml
<Literal>
  <ID>method</ID>
  <Default>Handler</Default>
</Literal>
```

이런 형태로 하고 code에 $method$를 쓰면 된다.

## 실행 

원하는 클래스에 가서 testt 타입후 탭을 두번누른다. 

그러면 다음 그림처럼 나온다. 

![]({{ site.baseurl }}/assets/2020-03-20-09-39-02.png)

이제 각각 이름들을 변경할수 있다. 변경후 tab을 누르면 아래쪽값을도 전부다 바뀌는것을 볼수 있다. 

기본값이 맞으면 탭을 눌러서 이동하면된다.

다 완료 되있으면 엔터를 치면 모든 코드가 적용된것을 알수 있다. 

생성된 코드는 위와 같다. 

```cs

[Fact]
public async Task Handler_DuplicateData_ThrowDuplicatedException()
{
  //Arrange
  var command = new CreateUserProfileCommand
  {
    Name = "Name",
  };

  //Act + Assert
  var sut = new CreateUserProfileCommand.CreateUserProfileCommandHandler(Context, Mapper, _userServiceMock.Object);
  var error = await Assert.ThrowsAsync<DuplicatedException>(async () => await sut.Handle(command, CancellationToken.None));
}
```


