---
layout: post
title: 'Automapper Mapfrom Vs Resolveusing' 
author: teamsmiley 
date: 2016-08-17
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: {program}
---
# Automapper에서 Mapfrom Vs Resolveusing 

AutoMapper에서 매핑하는 두가지 방법이 있다.

## ResolveUsing

```c#
Mapper.CreateMap<SourceType, DestType>().ForMember(d => d.DestPropX, o => o.ResolveUsing(s => s.SourcePropY));
```

## MapFrom

```c#
Mapper.CreateMap<SourceType, DestType>().ForMember(d => d.DestPropX, o => o.MapFrom(s => s.SourcePropY));
```

대부분의 경우에는 두개의 차이는 거의 없다. 

## 특별한 경우 

(오브젝트에 널이 있는경우 그리고 이런 객채의 리스트인 경우) 에는 고민해봐야할 부분이 있다.

MapFrom은 null 체크를 한다. 

예를 들면 

```c#
ForMember(d => d.DestPropX, o => o.MapFrom(s => s.Person.Address.State));
```

이런 코드에서 Person이나 Address 나 Status가 널일경우가 있다. 

오토매퍼가 자동으로 Null Reference 에러를 잡아서 숨겨준다. 좋아 보이지만 pay를 해야할것이 있다. 

만약 이런 오프젝트가 1000개이면 1000개를 다 널 익셉션이 나고 그걸 잡아줘야한다.  퍼포먼스가 많이 떨어진다.

그럴 경우에는 차라리 ResolveUsing을 사용하는게 나을듯 싶다.

```c#
ForMember(d => d.DestPropX, o => o.ResolveUsing(s => s.Person != null ? s.Person.Address.State : null ));
```

이게 훨씬 빠르다. 

원글은 <http://blog.travisgosselin.com/automapper-mapfrom-vs-resolveusing/> 이며 대충 읽어보고 핵심만 추려서 다시 설명을 한 것입니다.

혹시 잘못된부분이 있으면 teamsmiley@gmail.com으로 이메일 부탁 드립니다.












