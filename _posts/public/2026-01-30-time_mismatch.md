---
title: Time mismatch in new machine
author: rhea
date: 2026-01-30 20:00:00 +0900
categories: [troubleshooting]
tags: [linux, troubleshooting]
published: true
---

### **Problem**
새로운 머신에서 작업하다 보니 system 시간이 실제 시간과 맞지 않다는 것을 발견하였다.

```bash
[me@machine77 /home/me] timedatectl
               Local time: Thu 2026-01-29 23:45:54 EST
           Universal time: Fri 2026-01-30 04:45:54 UTC
                 RTC time: Fri 2026-01-30 04:45:54
                Time zone: America/New_York (EST, -0500)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

### **Cause**
machine77에 Rocky Linux 8.10 OS를 설치 후 timezone default 설정을 바꾸어 주지 않았다. 

### **Solution**
```bash
[me@machine77 /home/me] sudo timedatectl set-timezone Asia/Seoul
```
해결되었다!