---
layout: post
title: 'Automapper - 5 - Db Update 쉽게 하기' 
author: teamsmiley 
date: 2016-12-26
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: {automapper}
---
# Automapper에서 데이터 베이스 업데이트 코드 샘플 

기존 코드 

```cs
public IHttpActionResult PutUser(string userName, UserViewModel vm)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    var renderCoreUser = _unitOfWork.Users.SingleOrDefault(i => i.UserName == userName);
    User.AssignedNodeCnt = (int)vm.AssignedNodeCnt;
    User.FileServerId = (int)vm.FileServerId;
    User.NodeGroupId = (int)vm.NodeGroupId;
    User.PaymentType = vm.PaymentType;
    User.UnitPrice = (double)vm.UnitPrice;
    User.UserPriority = (int)vm.UserPriority;
    User.Location = vm.Location;

    _unitOfWork.Complete();

    var result = Mapper.Map<User, UserViewModel>(User);

    return Ok(result);

}
```

수정후 코드 

```cs
public IHttpActionResult PutUser(string userName, UserViewModel vm)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    var existUser = _unitOfWork.Users.SingleOrDefault(i => i.UserName == userName);
    var user = Mapper.Map<UserViewModel, User>(vm);  //viewmodel을 디비에서 가져온 정보에 업데이트한다.
   
    _unitOfWork.Complete(); //실제 디비업데이트한다.

    var result = Mapper.Map<User, UserViewModel>(existUser);

    return Ok(result);

}
```

매핑 하는 부분이 전부 사라졌다..코드가 많이 줄었다.

추가 

일반적으로 다음처럼 한다.

![]({{ site.baseurl }}/assets/automapper-db-update.png)

아주 간단해 졌다. 

혹시 잘못된부분이 있으면 teamsmiley@gmail.com으로 이메일 부탁 드립니다.

관련 글
 
AutoMapper - 1 <https://teamsmiley.github.io/2016/12/22/automapper-1>

AutoMapper - 2 <https://teamsmiley.github.io/2016/12/23/automapper-2> 

AutoMapper - 3 <https://teamsmiley.github.io/2016/12/23/automapper-3>

AutoMapper - 4 <https://teamsmiley.github.io/2016/12/23/automapper-4>

AutoMapper - 5 <https://teamsmiley.github.io/2016/12/26/automapper-5>

## 동영상 강의

http://channel9.msdn.com/posts/ASPNET-MVC-With-Community-Tools-Part-10-AutoMapper

http://www.dnrtv.com/?showNum=155

## 참고 사이트 

https://github.com/AutoMapper/AutoMapper/wiki

http://funnygangstar.tistory.com/entry/AutoMapper%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4%EA%B0%84-%EB%A7%B5%ED%95%91













