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
public IHttpActionResult PutUser(UserViewModel vm)
{
    //entity를 가져오고 
    var entity = _unitOfWork.Users.SingleOrDefault(i => i.UserName == vm.userName);
    //vm에서 값을 가져다 엔티티를 업데이트해준다. 
    entity.AssignedNodeCnt = (int)vm.AssignedNodeCnt;
    entity.FileServerId = (int)vm.FileServerId;
    entity.NodeGroupId = (int)vm.NodeGroupId;
    entity.PaymentType = vm.PaymentType;
    entity.UnitPrice = (double)vm.UnitPrice;
    entity.UserPriority = (int)vm.UserPriority;
    entity.Location = vm.Location;
   
    //entity가 업데이트 되었으므로 complete만 하면된다.
    _unitOfWork.Complete();
}
```

수정후 코드 

```cs
public IHttpActionResult PutUser(UserViewModel vm)
{
    // entity 가져오기 
    var entity =  _unitOfWork.Users.SingleOrDefault(i => i.UserName == vm.userName);

    //map
    //apply update 
    //map to back to entity 
    //  이 3개를 전부 다음코드 하나가 처리한다.
    Mapper.Map(vm, entity);
    //or
    Mapper.Map<UserViewModel,Users>(vm, entity);

   //entity가 업데이트 되었으므로 complete만 하면된다. 
   _unitOfWork.Complete(); //실제 디비업데이트한다.
}
```

매핑 하는 부분이 전부 사라졌다..코드가 많이 줄었다.


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













