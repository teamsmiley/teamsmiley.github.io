---
layout: post
title: 'Github commit message'
author: teamsmiley
date: 2021-07-01
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# Github commit message

## skip ci

Github Action을 커밋 메세지로 스킵하고 싶어졌다. 구지 ci가 필요가 없는 경우 빌드시간이 아까우니..

커밋 메세지에 다음을 사용하면 ci가 동작하지 않는다.

- [skip ci]
- [ci skip]
- [no ci]
- [skip actions]
- [actions skip]

<https://github.blog/changelog/2021-02-08-github-actions-skip-pull-request-and-push-workflows-with-skip-ci/>

## close issue

커밋메세지에 다음을 포함하면 이슈를 닫을수 있다.

- close
- closes
- closed
- fix
- fixes
- fixed
- resolve
- resolves
- resolved

ex)

- fixed #100
- 다른 프로젝트도 같이 : Fixes octo-org/octo-repo#100
- 여러 프로젝트 함께 : Resolves #10, resolves #123, resolves octo-org/octo-repo#100

<https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue#linking-a-pull-request-to-an-issue-using-a-keyword>
