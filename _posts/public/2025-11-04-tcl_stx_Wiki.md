---
title: Tcl syntax Wiki
author: rhea
date: 2025-11-04 16:30:00 +0900
categories: [tcl]
tags: [tcl, syntax, command]
published: true
---

### **Overview**
#### How Tcl(Tool Command Language) interpretation works
> Script를 Interpreter가 읽고, Command 단위로 해석하여 실행한다
- Script: 하나 이상의 Commands로 구성된 텍스트 파일 또는 Input stream
- Interpreter: Tcl 문법에 따라 Substitution(치환)과 Grouping(그룹화)을 수행하는 엔진
- Command: 공백으로 구분된 String List이며, 첫 번째 단어가 실행할 Command Name이 됨

#### Key Characteristics
- Everything is a String : 모든 Data types(숫자, 리스트, 코드 블록 등)를 내부적으로 String으로 처리하여 Data handling이 매우 유연함
- Command-centric : if, for과 같은 제어문조차 Reserved keywords가 아니라 하나의 Built-in Command로 구현되어 있음
- High Embeddability : C/C++ Application에 포함하기 매우 쉬워, EDA(하드웨어 설계 자동화) 및 네트워크 장비 제어 분야에서 표준처럼 사용됨

<br>

-----------------------

<br>

### **`source`**
- 지정된 Path의 외부 Tcl 스크립트 파일을 읽어 현재 Interpreter Session에서 즉시 Execute
- 파일 내의 모든 명령어를 현재 Context(Global Scope 또는 특정 Procedure 내부)에서 Sequentially execute
- 외부 파일에서 정의된 Variables나 Procedures(proc)를 현재 Session으로 Load하여 재사용하고 싶을 때 필수적임
- 파일 내에 return statement가 있다면 마지막 Execution Result를 Return하며, 오류 발생 시 실행을 즉시 Abort하고 Error를 Raise함
- Usage examples
  ```tcl
  # Load the common configuration file (config.tcl) into the current session
  source "scripts/config.tcl"

  # Execute an external script using an Absolute Path
  source "/home/user/eda/tools/lib_init.tcl"

  # Assign the execution result to a variable (if return is used in the script)
  set setup_status [source "project_setup.tcl"]
  ```
- <span style="color:DeepSkyblue"> **func eq!** </span> Lua의 `dofile`
- <span style="color:LightGreen"> **func like!** </span> Linux shell의 `source`
