---
title: Incorrect home mount after Linux home migration
author: rhea
date: 2026-02-25 16:00:00 +0900
categories: [troubleshooting]
tags: [linux, troubleshooting]
published: true
---

### **Problem**
machine77에 나의 home이 있다. 그러나 machine77에서 이전 머신인 machine00에 SSH 접속하면, `/home/me` dir이 machine77의 home이 아니라 예전 machine00 home이었다.

이전 머신 상태였다.
```bash
[me@machine00 /] df -h /home/me
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb        2.7T  2.4T  231G  92% /home/me
```

### **Cause**
home을 migrate했을 당시 machine00에 `umount`작업을 하지 않았었다.


### **Solution**
```bash
[me@machine00 /] sudo umount /home/me
umount: /home/me: target is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))
```
현재 시스템이나 어떤 프로그램이 `/home/me` dir을 사용 중이여서 강제로 연결을 끊을 수 없었다.
```bash
[me@machine00 /] sudo lsof /home/me
lsof: WARNING: can't stat() fuse.gvfsd-fuse file system /run/user/<my_uid>/gvfs
      Output information may be incomplete.
COMMAND     PID USER   FD   TYPE DEVICE  SIZE/OFF     NODE NAME
bash       1531 me  cwd    DIR   8,16         0 86925349 /home/me/work/<project_path_0> (deleted)
bash       4656 me  cwd    DIR   8,16      4096 96608292 /home/me/work/<project_path_1>
gnome-ses  9588 me  cwd    DIR   8,16      4096 86638593 /home/me
...
```
내가 이전에 띄워놓은 GUI 환경(GNOME, VNC, XRDP)과 시뮬레이션 프로세스들을 확인할 수 있었다.

```bash
sudo killall -u me
Connection to machine00 closed by remote host.
Connection to machine00 closed.
```
`/home/me`를 사용 중인 모든 프로그램을 죽였다. 현재 접속 중인 ssh 세션도 끊겼다.
```bash
[me@machine00 /] sudo umount /home/me
umount: /home/me: target is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))
```
다시 `sudo umount /home/me` 명령어를 입력하였지만 같은 결과가 나왔다. 아마 프로세스가 죽는 데 시간이 걸렸던 것으로 추정된다.

```bash
[me@machine00 /] sudo umount -fl /home/me
```
force & lazy option을 주어 unmount하였다.

```bash
[me@machine00 /home/me] systemctl status autofs
* autofs.service - Automounts filesystems on demand
   Loaded: loaded (/usr/lib/systemd/system/autofs.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2025-11-07 15:52:12 KST; 3 months 18 days ago
 Main PID: 2278 (automount)
    Tasks: 6
   Memory: 1.4M
   CGroup: /system.slice/autofs.service
           `-2278 /usr/sbin/automount --systemd-service --dont-check-daemon
```
이후 `/home/me` dir을 확인해보니 machine77의 home이다! 확인 결과 `autofs` 서비스 때문에 자동으로 mount가 된 것이었다.