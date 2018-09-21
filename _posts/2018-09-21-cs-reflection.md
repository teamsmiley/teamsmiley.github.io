---
layout: post
title: 'c# namesapce에 있는 하위 클래스 알아내기' 
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

