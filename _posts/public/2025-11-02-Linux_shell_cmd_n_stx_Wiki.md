---
title: Linux shell command & syntax Wiki
author: rhea
date: 2025-11-23 17:29:00 +0900
categories: [linux]
tags: [linux, syntax, command]
published: true
---

### **Overview**
#### How Linux commands works
> Terminal에서 받고, Shell이 해석해서, Kernel이 실행한다
- terminal : 사용자와 컴퓨터가 텍스트로 소통할 수 있게 해주는 I/O 환경 
- shell : terminal을 통해 입력된 command를 해석하여 OS kernel에게 전달하는 프로그램 
- kernel : 하드웨어를 제어하는 핵심 엔진

#### Command types
- Shell Built-in : shell에 내장된 명령어
- External : 별도의 실행 파일로 존재하는 명령어
- Alias/Function : 사용자가 정의한 단축 명령어

<br>

-----------------------

<br>

### **`top`**, **`htop`**
1. Real-time Resource Monitoring
    - CPU Usage: 전체 Core별 Utilization을 Bar graph로 표시. 특정 Core의 Bottleneck 확인 가능
    - Memory & Swap: RAM Free space 및 Swap memory status 실시간 Monitoring

2. Process Sorting & Searching
    - Sort : CPU(P), Memory(M), Runtime 순으로 나열하여 Resource Hog(과점유 프로세스) Identify
    - Filter : 특정 User(nathan)나 Program name(vsim)으로 Search 및 Filtering

3. Process Management (Kill & Priority)
    - Kill : Signal을 전송하여 Issue가 발생한 Process 즉시 Termination
    - Nice Adjustment : Priority를 Modify하여 Resource Allocation 최적화

4. Process Tree View
    - Hierarchy : F5 키로 Parent-Child 관계 확인 &rarr; Execution path 및 Zombie process의 Root cause 추적 용이

| function      | top (classic)         | htop (modern)
|---------------|-----------------------|---------------------------|
| Interface     | text-based (CLI)      | Graphical / Color UI      |
| Control       | shortcut keys only    | keyboard + mouse support  |
| Scrolling     | vertical only         | vertical & horizontal     |
| Process kill  | Manual PID Entry      | Point & Shoot (Selection) |


#### `top`
**Display Linux Processes**
- Default external command
- Usage examples
  ```bash
  # Update interval set to 1 second
  top -d 1

  # Monitor processes for a specific user
  top -u rhea

  # Batch mode: Save a single snapshot of top 10 processes to a file
  # -b: Enable Batch mode for file redirection
  # -n 1: Run for exactly 1 iteration and then exit automatically
  top -b -n 1 | head -n 20 > top_report.txt

  # Monitor specific PIDs only
  top -p 1024,2048
  ```

#### `htop`
***<u>H</u>isham's <u>TOP</u>***
- Non-default external command 
- Usage examples
  ```bash
  # Start htop in Tree view mode by default
  htop -t

  # Monitor specific user's processes with 1-second update interval
  htop -u rhea -d 10

  # Start with a specific sorting column (Memory usage)
  htop -s PERCENT_MEM

  # Monitor specific PIDs only
  htop -p 1234,5678
  ```

<br>

### **`kill`**, **`pkill`**
**Process terminate command**

#### `kill`
**Kill process by PID**
- Shell Built-in & Default external command
- PID(Process ID)를 직접 지정해서 termination
- Usage examples
- ex.
  ```bash
  # `SIGHUP` Reload configuration without restarting the service
  kill -1 1234
  kill -HUP 1234
  # `SIGTERM` Request a graceful shutdown (Default behavior)
  kill 1234
  kill -15 1234
  kill -TERM 1234
  # `SIGKILL` Immediate and forced termination by the kernel
  kill -9 1234
  kill -KILL 1234
  ```

#### `pkill`
***Process KILL*** by name
- Non-default external command
- PID를 몰라도 process 이름만으로 일괄 termination
- Usage examples
  ```bash
  # `pkill` Terminate all processes named 'python' at once
  pkill python

  # `pkill -f` Search and kill based on the full command line
  pkill -f "my_program.py"
  ```

<br>

### **`echo`**
**Echo string**
- Shell Built-in command
- 입력한 String이나 Shell Variables를 Terminal 화면에 Display
- <span style="color:DeepSkyblue"> **func eq!** </span> C의 `printf`
- Usage examples
  ```bash
  # Display a specific message or variable
  echo "Current fruit : "
  echo "Target Path : $PATH"
  ```

<br>

### **`rm`**
***<u>R</u>e<u>M</u>ove***
- Default external command
- Files나 Directories를 system에서 Delete
- <span style="color:tomato"> **Caution!** </span> 삭제된 데이터는 Trash Can으로 이동하지 않고 바로 Unlink되므로 복구가 어려움
- Usage examples
  ```bash
  # Remove a single file
  rm memo.txt

  # -r (Recursive): Remove a directory and all its contents
  rm -r backup_folder

  # -f (Force): Ignore nonexistent files and never prompt for confirmation
  rm -f temporary_log.log

  # Combine options for powerful deletion
  rm -rf old_project_dir
  ```

<br>

### **`mkdir`**
***<u>M</u>a<u>K</u>e <u>DIR</u>ectory***
- Default external command
- 새로운 Empty Directory를 Create
- Usage examples
  ```bash
  # Create a single directory
  mkdir new_project

  # -p (Parents): Create intermediate parent directories as needed
  # Useful for creating nested paths at once
  mkdir -p work/2024/reports/daily

  # -m (Mode): Set specific permissions (chmod style) during creation
  mkdir -m 755 public_html
  ```

<br>

### **`ln`, `ln -s`**
**Link management commands**

#### **`ln`**
**Hard Link**
- Default external command
- Target File과 동일한 Inode Number를 공유하는 새로운 이름을 Create
- Data Block을 직접 가리키는 포인터를 하나 더 생성 (Reference Count 증가)
- 원본 파일 이름을 rm으로 삭제해도, Hard Link가 남아있다면 Data는 삭제되지 않음
- Directory에는 생성 불가하며, 물리적으로 다른 Partition(File System) 간의 Link 생성 불가
- Usage examples
  ```bash
  # Create a hard link named 'hard_link' for 'original_file'
  # Both files will point to the same data on the disk
  ln original_file hard_link

  # Verify the link by checking the Inode Number
  # -i: Display the index number (Inode) of each file
  ls -li original_file hard_link

  # Update or overwrite an existing hard link
  # -f: Force removal of existing destination files
  ln -f source_file existing_link
  ```

#### **`ln -s`**
**Symbolic Link / Soft Link**
- Default external command
- Target File의 Path를 담고 있는 별도의 새로운 파일을 Create
- Windows의 Shortcut(바로가기)과 유사하며, 독자적인 Inode를 가짐
- Directory에 Link 생성이 가능하며, 다른 Partition이나 Network Drive 너머로도 Link 가능
- 원본 파일이 Move되거나 Delete되면 Link는 'Broken(빨간색 표시)' 상태가 되어 기능을 상실
- Usage examples
  ```bash
  # Create a Symbolic Link for a configuration file
  ln -s /etc/nginx/nginx.conf my_link.conf

  # Create a Symbolic Link for a Directory
  ln -s /var/log/apache2 ./logs
  ```

<br>

### **`rsync`**
***<u>R</u>emote <u>SYNC</u>***
- Default external command (Most Linux distributions)
- Local이나 Remote Host 간에 Files 및 Directories를 효율적으로 Synchronize
- Delta Transfer Algorithm으로 변경된 부분만 찾아내어 전송하므로 Network Bandwidth를 절약
- SSH를 통해 Data를 Encrypt하여 전송하며, 복잡한 백업 및 Mirroring 작업에 최적화
- Usage examples
  ```bash
  # Basic Archive Synchronization
  # -a (Archive): Preserve permissions, owners, and symbolic links
  # -v (Verbose): Display detailed transfer information
  # -z (Compress): Compress data during transfer to save bandwidth
  rsync -avz /src/directory/ /dest/directory/

  # Remote Transfer with Progress Monitoring
  # --progress: Show real-time transfer speed and remaining time
  # --exclude: Skip specific files or directories (e.g., 'work' folder)
  rsync -avz --progress --exclude='work' /home/me/ rhea@machine77:/home/me/

  # Mirroring with Deletion
  # --delete: Remove files in destination that no longer exist in source
  rsync -avz --delete /src/ /dest/

  # Dry Run for Safety
  # -n (Dry run): Perform a trial run without making any actual changes
  rsync -avzn --progress /src/ /dest/
  ```

<br>

### **`source`**
**Execute file in the current shell**
- Built-in shell command
- script 파일을 실행할 때 새로운 Subshell을 생성하지 않고, 현재 Shell 세션 내에서 직접 실행
- script 내부에서 정의된 변수, 함수, 환경 설정이 현재 세션에 즉시 반영됨 (영구 적용을 위해 .bashrc 수정 후 자주 사용)
- command 대신 dot(.) 기호를 사용하여 동일하게 수행 가능 (POSIX 표준)
- Usage examples
  ```bash
  # Load and reflect configurations, environments, or functions immediately
  # Import script content into the current shell context
  source ./my_scripts/useful_functions.sh

  # Alternative syntax (Dot operator)
  # Performs the same action using '.' (Note: A space is required after the dot)
  . ~/.profile
  ```
- <span style="color:PaleVioletRed"> **Pitfall!** </span> `./test.sh` vs `. ./test.sh`

  | Command | `./test.sh` (일반 실행) | `source ./test.sh` or `. ./test.sh` |
  | Proc scope | 새로운 Subshell(자식 쉘)을 띄워서 실행 | Current Shell에서 직접 실행 |
  | Env Var |	script 안에서 export 해도 현재 terminal엔 적용 X | script 안 export 등이 현재 terminal에 남음 |
  | Permission | 실행 권한(x)이 반드시 있어야 함 | 실행 권한 없어도 읽기 권한만 있으면 가능 |
  | Post-Exec | Subshell이 닫히면서 작업 내용(변수 등)이 증발 | 현재 terminal 환경이 script 내용대로 변경됨 |

<br>

-----------------------

<br>

### **`>`, `>>`**
- Shell Syntax
- Output Redirection Operators

#### **`>`**
Redirection - Overwrite
- 화면으로 출력될 Data Stream의 방향을 File로 전환하여 저장
- Target File이 존재하지 않으면 새롭게 Create하고, 이미 존재하면 overwrite하고 reset
- Usage examples
  ```bash
  # Initialize or replace file content
  ps -ef > current_processes.txt
  ```

#### **`>>`**
Redirection - Append
- 기존 File의 내용을 유지하면서 새로운 데이터를 EOF에 Append
- Usage examples
  ```bash
  # Add information without deleting existing data
  ls -R >> directory_structure.log
  ```
