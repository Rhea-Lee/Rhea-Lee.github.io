---
title: Oasys engine failed to initialize during Catapult synthesis
author: rhea
date: 2026-02-25 18:00:00 +0900
categories: [troubleshooting]
tags: [catapult, troubleshooting]
published: true
---

### **Problem**
catapult RTL 합성 시 다음과 같이 Oasys engine initialize가 fail되어 error log를 띄우는 현상이 있었다.
합성에는 지장이 없었다.

```bash
# Info: Starting transformation 'cluster' on solution '<design_name>.v1' (SOL-8)
# Info: Starting synthesis of module for CLU 'ROM_***_****_*****'... (CLUSTER-6)
# Error: Prototyping engine 'oasys' failed to initialize (LIB-204)
# Error: Prototyping engine 'oasys' failed to initialize (LIB-204)
# Module for the CLU 'ROM_***_****_*****' has been successfully synthesized (CLUSTER-9)
```

home migration 이전 machine에서는 이러한 error가 없었는데, home migration 이후 생긴 error이다.

### **Cause**
#### 1. library error
error log를 잘 보면, `LIB-204` 코드가 있다.

EDA 툴에서 `LIB`로 시작하는 error는 보통 라이브러리 파일을 로드하는 엔진이 파일이나 환경을 못 찾을 때 발생한다.
#### 2. home migration 전후 머신의 OS
- home migration 전 OS(machine00 OS) : Rocky Linux 8.10 (Green Obsidian)
- home migration 후 OS(machine77 OS) : CentOS Linux 7 (Core)

CentOS는 2014년에 나온 아주 오래된 표준을 따르고, Rocky는 훨씬 최신 표준을 따른다. 따라서 Rocky Linux에서 구형 EDA 툴을 돌릴 때 필수적인 하위 호환 패키지들이 존재한다.

Linux 세계에서 libnsl.so.1은 아주 유명한 골칫덩이라고 한다. 최신 OS로 넘어오면서 libnsl과 같은 낡은 library들을 기본값에서 제외해버렸는데, 반도체 설계 툴들은 여전히 10~20년 전 설계된 구조를 쓰다 보니 이런 library들이 없으면 엔진이 켜지지도 않고 죽는 것이다.

```bash
[me@machine00 /home/me] ls /usr/lib64/libnsl.so.1
/usr/lib64/libnsl.so.1
```
```bash
[me@machine77 /home/me/work/<project_path>] ls /usr/lib64/libnsl.so.1
ls: cannot access '/usr/lib64/libnsl.so.1': No such file or directory
```
역시나 machine00에는 있는 library가 machine77에는 없었다.

### **Solution**
libnsl은 network service library라고 한다. 이 library가 없어 오류가 발생하곤 한다는 정보를 듣고 설치하였다.

```bash
[me@machine77 /home/me/work/<project_path>] sudo dnf install libnsl
```

그러나 여전히 error log가 존재한다.

Catapult 설치 폴더 안에서 이름이 'oasys'로 시작하는 모든 file을 찾아보았다.

```bash
[me@machine77 /home/me/work/<project_path>] find $CTPT_HOME -name "oasys*" -type f
<ctpt_home_dir>/.../oasys
...
```

Oasys binary file(실제 실행 파일)에 대해 의존성을 체크했다.

```bash
[me@machine77 /home/me/work/<project_path>] ldd <ctpt_home_dir>/.../oasys | grep "not found"
	libncurses.so.5 => not found
```

위 file이 포함된 package를 설치하였다.

```bash
[me@machine77 /home/me/work/<project_path>] sudo dnf install ncurses-compat-libs
```

이후 다시 Catapult 합성해보니 error가 사라졌다!
