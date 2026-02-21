# 14 — Unix/Linux Commands (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🟡 Important  
> Covers: File Operations, Text Processing, Process Management, Networking, Shell Scripting

---

## 📌 Table of Contents

1. [File & Directory Commands](#1-file--directory-commands)
2. [Text Processing (grep, awk, sed)](#2-text-processing)
3. [File Permissions](#3-file-permissions)
4. [Process Management](#4-process-management)
5. [Disk & Memory](#5-disk--memory)
6. [Networking](#6-networking)
7. [Useful One-Liners](#7-useful-one-liners)
8. [Interview Questions & Answers](#8-interview-questions--answers)

---

## 1. File & Directory Commands

```bash
# Navigation
pwd                         # print working directory
cd /var/log                 # change directory
cd ~                        # home directory
cd -                        # previous directory

# Listing
ls -la                      # long format, all files (including hidden)
ls -lh                      # human-readable sizes
ls -lt                      # sort by modification time
ls -lS                      # sort by size

# File operations
cp file1.txt file2.txt      # copy
cp -r dir1/ dir2/           # copy directory recursively
mv old.txt new.txt          # move/rename
rm file.txt                 # delete
rm -rf directory/           # delete directory recursively (⚠️ careful!)
mkdir -p path/to/dir        # create nested directories
touch file.txt              # create empty file / update timestamp

# Viewing files
cat file.txt                # print entire file
head -20 file.txt           # first 20 lines
tail -20 file.txt           # last 20 lines
tail -f app.log             # follow (live) — great for monitoring logs
less file.txt               # paginated viewer (q to quit)
wc -l file.txt              # count lines
wc -w file.txt              # count words

# Finding files
find / -name "*.log"                    # find by name
find . -name "*.java" -mtime -7         # modified in last 7 days
find . -size +100M                      # files larger than 100MB
find . -type f -name "*.tmp" -delete    # find and delete

# which / whereis
which java                  # path to executable
whereis java                # binary, source, man page locations
```

---

## 2. Text Processing

### grep — Search text
```bash
grep "ERROR" app.log                    # find lines with ERROR
grep -i "error" app.log                 # case-insensitive
grep -r "TODO" src/                     # recursive search in directory
grep -n "Exception" app.log             # show line numbers
grep -c "ERROR" app.log                 # count matching lines
grep -v "DEBUG" app.log                 # invert (exclude DEBUG lines)
grep -A 3 "ERROR" app.log              # show 3 lines AFTER match
grep -B 2 "ERROR" app.log              # show 2 lines BEFORE match
grep -E "ERROR|WARN" app.log           # regex (extended grep)
grep -l "password" *.properties         # list files containing pattern

# Pipe with grep
cat app.log | grep "ERROR" | grep "2026-02"
ps aux | grep java
```

### awk — Column processing
```bash
# Print specific columns
awk '{print $1, $3}' file.txt           # print columns 1 and 3

# With delimiter
awk -F',' '{print $2}' data.csv         # CSV second column
awk -F':' '{print $1}' /etc/passwd      # usernames

# Conditional
awk '$3 > 50000 {print $1, $3}' employees.txt  # salary > 50000

# Sum a column
awk '{sum += $3} END {print sum}' data.txt

# Count unique values
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log
```

### sed — Stream editor (find & replace)
```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace ALL occurrences
sed 's/old/new/g' file.txt

# Replace in-place
sed -i 's/old/new/g' file.txt

# Delete lines
sed '/DEBUG/d' app.log                  # delete lines containing DEBUG
sed '1,5d' file.txt                     # delete lines 1-5

# Print specific lines
sed -n '10,20p' file.txt                # print lines 10-20
```

### sort & uniq
```bash
sort file.txt                           # sort alphabetically
sort -n file.txt                        # numeric sort
sort -r file.txt                        # reverse sort
sort -t',' -k2 file.csv                # sort by 2nd column (CSV)

uniq file.txt                           # remove adjacent duplicates
sort file.txt | uniq                    # remove all duplicates
sort file.txt | uniq -c                 # count occurrences
sort file.txt | uniq -d                 # show only duplicates
```

### cut & tr
```bash
cut -d',' -f1,3 data.csv               # extract fields 1 and 3
cut -c1-10 file.txt                     # extract characters 1-10

echo "Hello World" | tr 'a-z' 'A-Z'    # to uppercase
echo "a:b:c" | tr ':' '\n'             # replace delimiter with newline
```

---

## 3. File Permissions

```bash
# Permission format: rwxrwxrwx (owner/group/others)
# r=4, w=2, x=1

chmod 755 script.sh                     # rwxr-xr-x
chmod 644 file.txt                      # rw-r--r--
chmod +x script.sh                      # add execute
chmod -w file.txt                       # remove write
chmod u+x,g+r file.txt                  # user: +execute, group: +read

chown user:group file.txt               # change owner
chown -R user:group directory/          # recursive
```

| Number | Permission |
|--------|-----------|
| 7 | rwx (read + write + execute) |
| 6 | rw- (read + write) |
| 5 | r-x (read + execute) |
| 4 | r-- (read only) |
| 0 | --- (no access) |

---

## 4. Process Management

```bash
ps aux                                  # all processes
ps aux | grep java                      # find Java processes
ps -ef | grep spring                    # alternative format

top                                     # real-time process monitor
htop                                    # better interactive monitor

kill PID                                # graceful stop (SIGTERM)
kill -9 PID                             # force kill (SIGKILL)
killall java                            # kill all Java processes

# Background processes
command &                               # run in background
nohup command &                         # run in background, survive logout
jobs                                    # list background jobs
fg %1                                   # bring job to foreground
bg %1                                   # send to background

# Cron jobs
crontab -e                              # edit cron jobs
crontab -l                              # list cron jobs

# Cron syntax: minute hour day month weekday
# */5 * * * * /path/script.sh           # every 5 minutes
# 0 2 * * * /path/backup.sh            # daily at 2 AM
# 0 0 * * 0 /path/weekly.sh            # weekly on Sunday
```

---

## 5. Disk & Memory

```bash
df -h                                   # disk usage (human-readable)
du -sh *                                # size of each item in current dir
du -sh /var/log                         # size of specific directory
free -h                                 # RAM usage

# Find large files
find / -size +500M -type f 2>/dev/null
du -sh * | sort -rh | head -10          # top 10 largest items
```

---

## 6. Networking

```bash
curl http://localhost:8080/api/health    # HTTP GET
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"John"}' http://localhost:8080/api/users

ping google.com                          # test connectivity
netstat -tlnp                           # listening ports
ss -tlnp                                # modern alternative to netstat
lsof -i :8080                           # what's using port 8080
wget https://example.com/file.zip       # download file
scp file.txt user@host:/path            # secure copy to remote
ssh user@host                           # SSH into remote server

# Check if port is open
telnet host 8080
nc -zv host 8080
```

---

## 7. Useful One-Liners

```bash
# Top 10 most frequent errors
grep "ERROR" app.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head -10

# Count requests per status code in access log
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Find and kill process on specific port
kill $(lsof -t -i:8080)

# Monitor log file for specific pattern
tail -f app.log | grep --line-buffered "ERROR"

# Disk usage by directory, sorted
du -sh */ | sort -rh

# Find recently modified files
find . -mmin -30 -type f                # modified in last 30 minutes

# Replace text in all files
find . -name "*.java" -exec sed -i 's/oldText/newText/g' {} +

# Archive and compress
tar -czf backup.tar.gz /path/to/dir     # create
tar -xzf backup.tar.gz                  # extract
```

---

## 8. Interview Questions & Answers

### Q1: How do you find a specific text in all files in a directory?
**A:** `grep -r "searchText" /path/to/dir` or `grep -rl "text" .` to list only filenames.

### Q2: How do you find which process is using a specific port?
**A:** `lsof -i :8080` or `netstat -tlnp | grep 8080` or `ss -tlnp | grep 8080`.

### Q3: What is the difference between `>` and `>>`?
**A:** `>` overwrites the file. `>>` appends to the file. Example: `echo "log" >> app.log`.

### Q4: How do you check disk space?
**A:** `df -h` for filesystem usage, `du -sh *` for directory sizes.

### Q5: How do you run a process in the background that survives logout?
**A:** `nohup command &` or use `screen`/`tmux` for persistent sessions.

### Q6: What does `chmod 755` mean?
**A:** Owner: rwx (7), Group: r-x (5), Others: r-x (5). Owner can read, write, execute. Everyone else can read and execute but not write.

### Q7: How do you schedule a recurring task in Linux?
**A:** Use `crontab -e` to add a cron job. Syntax: `minute hour day month weekday command`. Example: `0 2 * * * /path/backup.sh` runs daily at 2 AM.

---

> **Next:** [15_DSA_Interview_Problems.md](15_DSA_Interview_Problems.md) →
