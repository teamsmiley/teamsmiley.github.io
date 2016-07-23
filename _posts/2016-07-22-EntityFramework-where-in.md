---
layout: post
title: '엔티티 프레임워크 WHERE IN' 
author: teamsmiley 
date: 2016-07-22
tags: [entity framework]
image: /files/covers/blog.jpg
category: {programe}
---

# entity framework 에서 where in 사용법

코드 리뷰를 하다보니 이상한 코드가 발견되었다.

```c#

public IEnumerable<Job> GetAbleToKillingJob(List<int> jobIds)
{
    List<Job> ableJob = new List<Job>();

    foreach (var jobId in jobIds)
    {
        var temp = RendercoreContext.Jobs.SingleOrDefault(j => j.JobId.Equals(jobId));
        ableJob.Add(temp);
    }

    ableJob = ableJob.Where(j => j.Status == (int)JobHelper.Status.JOB_WAITING || j.Status == (int)JobHelper.Status.JOB_RUNNING).ToList();

    return ableJob;
}
```

list에 들어있는 id를 가지고 루프를 돌면서 가져온후에 다시 리스트에 담는다.

이 리스트를 리턴한다..

수정을 해본 코드

```c#

public IEnumerable<Job> GetAbleToResumeJob(List<int> jobIds)
{
    return RendercoreContext.Jobs
        .Where(j => jobIds.Contains(j.JobId))
        .Where(j => j.Status == (int)JobHelper.Status.JOB_PAUSE).ToList();
}

```

Contains 함수를 사용하여 다음처럼 하니까 잘 된다.



