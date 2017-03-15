--- 
layout: post 
title: "2002년 아마존 회장의 이메일" 
date: 2017-03-15 00:00  
author: teamsmiley 
tags: [ReadMe]
image: /files/covers/blog.jpg
category: {ReadMe}
---

# 2002년 아마존 회장의 이메일 Jeff Bezos 

대략 정리하면

아마존의 모슨 서비스는 규격화된 인터페이스를 제공해야 하고, 이것을 외부에서 이용할 수 있도록 해야 한다.

우리 회사도 이래야겟다.

> In 2002, Jeff Bezos (CEO of Amazon), insisted that all Amazon services be built in a way that they couldeasily communicate with each other over Web protocol, and he issued a mandate requiring all teams to expose their data and functionality through services interfaces.

>This was his mandate:

1. All teams will henceforth expose their data and functionality through service interfaces.
1. Teams must communicate with each other through these interfaces.
1. There will be no other form of interprocess communication allowed: no direct linking, no direct reads of another team’s data store, no shared-memory model, no back-doors whatsoever. The only communication allowed is via service interface calls over the network.
1. It doesn’t matter what technology they use. HTTP, Corba, Pubsub, custom protocols — doesn’t matter.
1. All service interfaces, without exception, must be designed from the ground up to be externalizable. That is to say, the team must plan and design to be able to expose the interface to developers in the outside world. No exceptions.
1. Anyone who doesn’t do this will be fired.
1. Thank you; have a nice day!

>Jeff Bezos understood that in order for his company to be successful, he had to switch focus from creating a “perfect product” to creating a perfect platform for that product.

>As a result of that, Amazon has built the most widely used cloud services in the world (AWS).
 

