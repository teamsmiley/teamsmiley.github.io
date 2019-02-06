# SSH Default User 세팅 

맥을 사용하면서 기본 유저이름을 rrrrr으로 쓰다보니 ssh로 접근할 때 마다 username을 적어 줘야해서 불편햇다.

예를 들면 
```
ssh root@my-server 
```

대부분 root를 쓰기때문에 혹시 이런게 잇을가 하고 찾아봤더니 있엇다.

man ssh_config를 하면 전체 옵션을 볼수 있다. 

내가 필요한 부분은 다음과 같다.

vi ~/.ssh/config
```
Host *
    User root
```

이렇게 해주면 다음부터는 root 사용해서 더이상 타입할 필요가 없다.

```
ssh my-server
```
하면 root가 붙어서 자동 로그인된다.

호스트 마다 다른경우는 아래처럼 하면될듯..테스트는 안해봣음.
```
Host aaa
    HostName aaa.net
    User rrrrr

Host bbb
    HostName bbb.net
    User root
```


