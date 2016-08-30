---
layout: post
title: 'C# - Gmail Sender' 
author: teamsmiley 
date: 2016-08-29
tags: [iis]
image: /files/covers/blog.jpg
category: {program}
---

#  C#으로 지메일 gmail 보내기 

메세지를 만들어서 다음처럼 하면 간단하게 보낼수가 있다. 


```cs
using (SmtpClient client = new SmtpClient
{
    EnableSsl = true,
    Host = "smtp.gmail.com",
    Port = 587,
    Credentials = new NetworkCredential("Your Id", "Your Pass")
})
{
    MailAddress from = new MailAddress("FromEmailAddress","FromName",System.Text.Encoding.UTF8);
    MailAddress to = new MailAddress("ToEmailAddress");
                       
    MailMessage message = new MailMessage(from, to);

    message.Body = "This is a test e-mail message sent by an application. ";
    message.BodyEncoding = System.Text.Encoding.UTF8;
    message.Subject = "subject " ;
    message.SubjectEncoding = System.Text.Encoding.UTF8;

    client.Send(message);
}
```

이메일 설정이 여기에 들어가도 되지만 보통 web.config에 설정해두고 사용한다. 

web.conf에 다음을 추가한다. 

```xml
<system.net>
    <mailSettings>
      <smtp deliveryMethod ="Network">
        <network host="smtp.gmail.com" 
                 port="587" 
                 userName="Gmail ID"
                 password="Gmail Pass" 
                 enableSsl="true" />
      </smtp>
    </mailSettings>
</system.net>
```

이제 실제 코드는 다음처럼 변경된다. 

```cs


MailAddress from = new MailAddress("FromEmailAddress","FromName",System.Text.Encoding.UTF8);
MailAddress to = new MailAddress("ToEmailAddress");
                    
MailMessage message = new MailMessage(from, to);

message.Body = "This is a test e-mail message sent by an application. ";
message.BodyEncoding = System.Text.Encoding.UTF8;
message.Subject = "subject " ;
message.SubjectEncoding = System.Text.Encoding.UTF8;

SmtpClient client = new SmtpClient();
client.Send(message);

```





