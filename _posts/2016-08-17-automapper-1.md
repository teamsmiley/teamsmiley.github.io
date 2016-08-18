---
layout: post
title: 'Automapper에서 매핑하기' 
author: teamsmiley 
date: 2016-08-17
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: {program}
---
# Automapper에서 매핑하기 
```c#
class App 
{
	public string Name;
} 

class Version
{
	public string Name;
	public  App App { get; set; }
}

class Plugin 
{
	public string Name
	public Version Version;
}
```

추가 설명을 하면 plugin  이 version을 가지고 있고 versoin이 app을 가지고 있는 구조이다.

이와 같을때 뷰모델 

```c#
public class ViewModel 
{
	public string Name { get; set; }
	public string A_Name { get; set; }
	public string V_Name { get; set; }        
}
```

여기에 매핑을 하려면 

```c#
cfg.CreateMap<Plugin, ViewModel>()
	.ForMember(dest => dest.A_Name, opt => opt.MapFrom(src => src.Version.App.Name))
	.ForMember(dest => dest.V_Name, opt => opt.MapFrom(src => src.Version.Name))
	.ForMember(dest => dest.Name , opt => opt.MapFrom(src => src.Name)
	;
```

이렇게 하면 된다.

 






