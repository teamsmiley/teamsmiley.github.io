---
layout: post
title: 'c# name sapce 하위 클래스들 알아내기' 
author: teamsmiley
date: 2018-09-21
tags: [c#]]
image: /files/covers/blog.jpg
category: {c#}
---

# c# namesapce에 있는 하위 클래스 알아내기

## 코드를 보자.

```cs
using System;
using System.Linq;
using System.Reflection;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            string nspace = "ConsoleApp1";

            var q = from t in Assembly.GetExecutingAssembly().GetTypes()
                    where t.IsClass && t.Namespace == nspace
                    select t;
            q.ToList().ForEach(t => Console.WriteLine(t.Name));
        }
    }

    class AAA
    {

    }

    class BBB
    {

    }
}
```

AAA BBB 가 찍히는것을 볼수 있다.

## 배운점

질문을 잘 정리하자. 그럼 빠른 결론에 도달하거나 정리하면서 해결이 되버리기도 한다.

