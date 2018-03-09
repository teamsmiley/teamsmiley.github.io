# standup-slack-bot 

[https://github.com/18F/standup-slack-bot] 사용

슬랙에 로그인한 상태로 https://api.slack.com/ 로 들어가서 start build버튼을 누른다.

create new 버튼 클릭한다.

![]({{ site.baseurl }}/assets/create-app.png)

![]({{ site.baseurl }}/assets/create-app-bot.png)

bot을 선택한다. 

![]({{ site.baseurl }}/assets/create-app-bot-user.png)

![]({{ site.baseurl }}/assets/create-app-bot-user-1.png)

![]({{ site.baseurl }}/assets/create-app-oauth.png)

![]({{ site.baseurl }}/assets/create-app-oauth-1.png)

키를 복사해둔다.

https://rendercorelab.slack.com/apps/manage

에서 approve 

![]({{ site.baseurl }}/assets/aprove-app.png)


![]({{ site.baseurl }}/assets/bot.png)

이렇게 되면 성공

이제 서버에서 

git clone https://github.com/18F/standup-slack-bot.git

cd standup-slack-bot

복사해둔 값을 넣어준다. .env이다.

```bash 
echo "SLACK_TOKEN=xoxb-325554995043-Ex5KQZf5VZn32HRpTV0EKhCr" > .env
echo "TIMEZONE=Asia/Seoul" >> .env
```

docker-compose up -d

서버 설치 완료 

이제 슬랙에서 특정 채널에 들어가서 세팅을 한다. 

특정 채널에 봇을 초대를 하고 월요일부터 금요일가지 9:00 시에 스탠드업 미팅을 한다. 

```
/invite @bot 
@bot reminder 10 
@bot create standup 09:00am M T W Th F
@bot enable updates
@bot audience @here

@bot when
@bot interview
@bot report
```

이제 등록해보자. bot에게 dm을 보내자.
```
standup #channelname
Y: yesterday's info
T: today's info
B: blockers
G: goals'

standup #korea_only T: cleanup all open issue
```
Y T B G 다 옵셔널이다. 꼭 다 써야하는건 아님 그러나  채널 이름은 꼭 써야함.

채널에서 지울려면 @bot remove standup




