---
layout: post
title: Apache Directory listing 제거
date: 2016-10-29 09:04
categories: Etc
---

Apache설치한경로에서  httpd.conf를 수정한다.

> /etc/httpd/conf/httpd.conf	(RPM 설치)
> 
> /usr/local/apach/conf/httpd.conf	(tar 설치)


**Options Indexes FollowSymLinks Includes**

또는

**Options All Indexes FollowSymLinks MultiViews**

위 두문구중에서 하나는 있을 것이다.

거기서 `Indexes`구문을 제거하고 아래와 같이 수정하고 저장한다.


**Options FollowSymLinks Includes**

또는

**Options All FollowSymLinks MultiViews**

그리고 apache를 재시작하면 끝~!!

