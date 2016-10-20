---
layout: post
category: blog
published: false
title: Android Thread Tips
---
1. 너무 많은 백그라운드 스레드 생성시 UI성능이 오히려 감소

응용 프로그램에 의해 생성된 스레드는 UI스레드와 동일한 우선 순위 가고 UI스레드와 같은 컨트롤 그룹의 구성원이 되어 동등 조건에서 프로세스 할당을 경쟁함.

-> Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND) 적용시 응용프로그램의 프로세스 수준에서 분리되어 백그라운드 그룹에 속하게 한다.


