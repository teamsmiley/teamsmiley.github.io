---
layout: post
title: "bash for loop"
author: teamsmiley
date: 2021-03-08
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# bash for loop

bash를 사용하다보면 루프를 사용하고 싶은때가 있다. 할때마자 찾아봐서 정리해보았다.

루프를 돌아야할 문자열이 있다.

```
a
b
c
d
```

이 문자열을 변수로 만들어야한다.

```bash
declare -a arr=(
a
b
c
d
)
```

이제 이 arr변수를 돌면서 하고싶은 일을 하면된다.

```bash
for i in "${arr[@]}"
do
  #해야할일들
  echo "$i"
  kcn "$i"
  kubectl delete secret regcred
done
```

이러면 a b c d를 루프 돌면서 하고싶은일을 한다.

너무 간단하지만 매번 찾아보게되서 적어보았다. 

"a" "b" 이런상태로도 가능하지만 일을 하다보니 문자열별로 줄바꿈이 있는게 더 많아서 위처럼 줄바꿈을 사용했고 " 이것이 없어도 잘되서 꼭 넣지 않았다.
