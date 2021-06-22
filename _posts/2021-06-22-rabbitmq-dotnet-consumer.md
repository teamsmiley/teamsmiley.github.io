---
layout: post
title: 'RabbitMQ 와 dotnet Worker Service Project'
author: teamsmiley
date: 2021-06-22
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# RabbitMQ 와 dotnet Worker Service Project

rabbitmq를 쓸 일이 있어서 정리해 보았다. 여기서는 consumer만 만들어서 처리하는 과정을 정리

## rabbitmq를 실행

```bash
docker run --rm --hostname rabbitmq --name rabbitmq -p 30000:15672 -p 5672:5672 rabbitmq:3-management
```

localhost:30000으로 접속해서 확인해본다. 기본 아이디 비번은 guest/guest이다. 

## Worker Service Project 생성

dotnet worker service template을 사용하여 프로젝트를 생성한다.

worker service에 대한 설명은 생략한다. rabbitmq와 연동만 관심있음 

## nuget 설치

https://www.nuget.org/packages/RabbitMQ.Client

```bash
dotnet add package RabbitMQ.Client --version 6.2.1
```

## Work 설정

```cs
public class EmainSenderWorker : BackgroundService
{
    private readonly ILogger<Worker> _logger;
    private ConnectionFactory _connectionFactory;
    private IConnection _connection;
    private IModel _channel;
    private const string _queueName = "email";

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
    }

    public override Task StartAsync(CancellationToken cancellationToken)
    {
        _connectionFactory = new ConnectionFactory
        {
          HostName = "localhost",
          UserName = "guest",
          Password = "guest",
          DispatchConsumersAsync = true
        };

        _connection = _connectionFactory.CreateConnection();
        _channel = _connection.CreateModel();

        _channel.QueueDeclare(queue: _queueName,
                            durable: false,
                            exclusive: false,
                            autoDelete: false,
                            arguments: null);

        _channel.BasicQos(0, 1, false);
        _logger.LogInformation($"Queue [{_queueName}] is waiting for messages.");

        return base.StartAsync(cancellationToken);
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // 이부분에서 처리를 해야한다.
    }

    public override async Task StopAsync(CancellationToken cancellationToken)
    {
        await base.StopAsync(cancellationToken);
        _connection.Close();
        _logger.LogInformation("RabbitMQ connection is closed.");
    }
}
```

StartAsync, StopAsync 두개 부분만 일단 구현했다. 이제 ExecuteAsync 구현해보자.

```cs
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
  _logger.LogInformation("Email Sender Start...");

  stoppingToken.ThrowIfCancellationRequested();

  var consumer = new AsyncEventingBasicConsumer(_channel);
  consumer.Received += async (bc, ea) =>
  {
    try
    {
      var message = Encoding.UTF8.GetString(ea.Body.ToArray());
      var email = JsonConvert.DeserializeObject<EmailMessage>(message);

      _logger.LogInformation($"Processing title: '{email.Title}'.");

      EmailSender sender = new EmailSender
      {
        EmailHost = "smtp.gmail.com",
        EmailPort = 587,
        EmailUserName = "yourid",
        EmailPassword = "your password",
        EmailEnableSSL = true
      };
      await sender.SendEmailAsync(email);

      _channel.BasicAck(ea.DeliveryTag, false);
    }
    catch (JsonException ex)
    {
      _logger.LogError($"JSON Parse Error {ex.Message}");
      _channel.BasicNack(ea.DeliveryTag, false, false);
    }
    catch (Exception e)
    {
      _logger.LogError(default, e, e.Message);
      _channel.BasicNack(ea.DeliveryTag, false, false);
    }
  };

  _channel.BasicConsume(queue: _queueName, autoAck: false, consumer: consumer);

  await Task.CompletedTask;
}
```

데이터 전달용으로 다음 사용

EmailMessage.cs

```cs
public class EmailMessage
{
  public string Title { get; set; } = "";
  public string Content { get; set; } = "";
  public string EmailTo { get; set; } = "";
  public List<string>? EmailCC { get; set; }
  public byte[]? Attachment { get; set; }
  public string? AttachmentName { get; set; }
}
```
 
EmailSender.cs

```cs
public class EmailSender
{
  public string EmailHost = "";
  public int EmailPort = 587; //gmail
  public string EmailUserName = "";
  public string EmailPassword = "";
  public bool EmailEnableSSL = true;

  public EmailSender()
  {

  }

  public async Task SendEmailAsync(EmailMessage message)
  {
    try
    {
      var client = new SmtpClient(EmailHost, EmailPort)
      {
        Credentials = new NetworkCredential(EmailUserName, EmailPassword),
        EnableSsl = EmailEnableSSL
      };

      var mail = new MailMessage() { IsBodyHtml = true };
      mail.From = new MailAddress(EmailUserName);
      mail.Subject = message.Title;
      mail.Body = message.Content;

      foreach (var item in message.EmailTo.Split(new[] { ";", "," }, StringSplitOptions.RemoveEmptyEntries))
      {
        mail.To.Add(item);
      }

      if (message.EmailCC != null)
      {
        foreach (var item in message.EmailCC)
        {
          mail.CC.Add(item);
        }
      }

      if (message.Attachment != null && message.AttachmentName != null)
      {
        var stream = new MemoryStream(message.Attachment);
        mail.Attachments.Add(new Attachment(stream, message.AttachmentName));
      }

      await client.SendMailAsync(mail);

    }
    catch (Exception ex)
    {
      throw ex;
    }
  }
}
```

## 테스트

실행하고 나면 대기하는것을 볼수 있다.

이제 웹사이트에서 메세지를 추가해보자.



![]({{ site_baseurl }}/assets/2021-06-22-rabbitmq-dotnet-consumer/2021-06-22-08-32-05.png)

consulers에 하나가 붙어잇는것을 볼수 있고 

publish message에서 메세지를 하나 넣어보자.

```json
{
  title: "test",
  content: "test",
  emailTo: "support@xxxx.com",
  emailCC: ["brian@xxxx.com"],
}
```

메세지를 publish 하자 마자 처리되는것을 볼수 있다.

![]({{ site_baseurl }}/assets/2021-06-22-rabbitmq-dotnet-consumer/2021-06-22-08-33-56.png)

이메일도 잘 들어왔다.

![]({{ site_baseurl }}/assets/2021-06-22-rabbitmq-dotnet-consumer/2021-06-22-08-34-29.png)

참고로 dockerfile

```text
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
WORKDIR /src
COPY ["EmailSender.csproj", ""]
RUN dotnet restore "./EmailSender.csproj"
COPY . .
WORKDIR /src
RUN dotnet publish "EmailSender.csproj" -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/runtime:5.0
WORKDIR /app
COPY --from=build-env /app/publish .
ENTRYPOINT ["dotnet", "EmailSender.dll"]
```

끝