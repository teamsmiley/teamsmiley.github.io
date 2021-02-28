---
layout: post
title: "dotnet 5 console program 배포"
author: teamsmiley
date: 2021-02-28
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# dotnet 5 console program 배포

닷넷 으로 exe를 만들엇는데 배포를 해야하는데 어떻게 해야하는지 몰라서 찾아보았다.

## 기본 publish

기본적으로 다음 코드를 실행한다.

```bash
dotnet publish -c Release
```

## 프레임워크 종속 배포

`--self-contained` 옵션을 사용하면된다.

`--self-contained false`를 사용하면 dotnet runtime 을 설치한 후에 exe을 실행할수 있다.

dotnet <PROJECT-FILE>.dll 로 실행

## 프레임워크 종속 실행 파일

.NET 5(및 .NET Core 3.1) SDK CLI의 경우 FDE(프레임워크 종속 실행 파일)가 기본 dotnet publish 명령의 기본 모드입니다.

프레임워크별로 실행 호스트가 만들어진다. 윈도우의 경우 <PROJECT-FILE>.exe 등이 된다.

dotnet <PROJECT-FILE>.dll을 호출하는 대신 exe를 실행하면 exe가 dotnet <PROJECT-FILE>.dll 를 호출해준다.

-r을 사용하여 rid를 넣어주면 된다. rid 는 다음에서 확인한다.

- <https://docs.microsoft.com/ko-kr/dotnet/core/rid-catalog>
- <https://github.com/dotnet/runtime/blob/master/src/libraries/Microsoft.NETCore.Platforms/pkg/runtime.json>

```bash
dotnet publish -c Release -r <RID> --self-contained false
```

Windows RID

- 이식 가능
  - win-x64
  - win-x86
  - win-arm
  - win-arm64
- Windows 7 / Windows Server 2008 R2
  - win7-x64
  - win7-x86
- Windows 8.1 / Windows Server 2012 R2
  - win81-x64
  - win81-x86
  - win81-arm
- Windows 10 / Windows Server 2016
  - win10-x64
  - win10-x86
  - win10-arm
  - win10-arm64

Linux RID

- 이식 가능
  - linux-x64(CentOS, Debian, Fedora, Ubuntu 및 파생 버전을 비롯한 대부분의 데스크톱 배포)
  - linux-musl-x64 (Alpine Linux와 같이 musl을 사용하는 간단한 배포)
  - linux-arm(Raspberry Pi 모델 2+의 Raspbian과 같이 ARM에서 실행되는 Linux 배포)
  - linux-arm64(Raspberry Pi 모델 3+의 Ubuntu Server 64비트와 같이 64비트 ARM에서 실행되는 Linux 배포)
- Red Hat Enterprise Linux
  - rhel-x64 (버전 6보다 상위 RHEL의 경우 linux-x64로 대체됨)
  - rhel.6-x64

## 자체 포함 배포

위처럼 하면 여러개의 dll과 폴더가 나온다. 파일이 너무 많아서 귀찮다. 단일 파일로 플랫폼별 실행 파일을 만든다.

`--self-contained true` 이 스위치는 .NET Core SDK에 실행 파일을 자체 포함 배포 생성하도록 지시합니다.

```bash
dotnet publish -c Release -r <RID> --self-contained true
```

## 단일 파일 배포 및 실행 파일

모든 애플리케이션 종속 파일을 하나의 이진 파일로 묶으면 애플리케이션 개발자가 애플리케이션을 단일 파일로 배포할 수 있습니다.

### cli

`-p:PublishSingleFile=true` 이걸 이용해서 만들면된다.

```bash
dotnet publish -r win-x64 -p:PublishSingleFile=true --self-contained true
dotnet publish -r linux-x64 -p:PublishSingleFile=true --self-contained false
```

### Project Properties

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
    <DebugType>embedded</DebugType>
  </PropertyGroup>
</Project>
```

PDB 파일이 따로 만들어진다. 이걸 exe파일에 포함하려면

```xml
<PropertyGroup>
  <DebugType>embedded</DebugType>
</PropertyGroup>
```

이렇게 해주면 됨.

## ReadyToRun

ReadyToRun(R2R) 형식으로 애플리케이션 어셈블리를 컴파일하면 .NET Core 애플리케이션의 시작 시간과 대기 시간을 개선할 수 있습니다. R2R은 AOT(Ahead-Of-Time) 컴파일 양식입니다.

실험적이다.

## 실행 파일 트리밍

트리밍은 게시된 자체 포함 애플리케이션에만 사용할 수 있습니다.

```
/p:PublishTrimmed=true
```

또는 설정에서

```xml
<PropertyGroup>
  <RuntimeIdentifier>win-x64</RuntimeIdentifier>
  <PublishTrimmed>true</PublishTrimmed>
</PropertyGroup>
```
