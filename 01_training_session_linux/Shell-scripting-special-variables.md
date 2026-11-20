# Shell Scripting — Special Variables

Shell scripting lets you automate Linux commands by putting them into a file and running that file.

Example:

```bash
#!/bin/bash

echo "Hello, Linux"
```

Run it:

```bash
chmod +x script.sh
./script.sh
```

---

## 1. What Are Special Variables?

Special variables are built-in Bash variables that provide information about:

- The script name
- Arguments passed to the script
- Number of arguments
- Exit status of a command
- Process ID
- Background processes

They start with `$`.

---

## 2. Important Special Variables

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1` to `$9` | First to ninth argument |
| `${10}` | Tenth argument and beyond |
| `$#` | Number of arguments |
| `$@` | All arguments as separate values |
| `$*` | All arguments as one value |
| `$?` | Exit status of the previous command |
| `$$` | Process ID of the current script |
| `$!` | Process ID of the last background command |
| `$_` | Last argument of the previous command |

---

## 3. `$0` — Script Name

Create `script-info.sh`:

```bash
#!/bin/bash

echo "Script name: $0"
```

Run:

```bash
./script-info.sh
```

Output:

```text
Script name: ./script-info.sh
```

Use case: display proper usage instructions.

```bash
echo "Usage: $0 <filename>"
```

---

## 4. `$1`, `$2`, `$3` — Positional Arguments

Create `greet.sh`:

```bash
#!/bin/bash

echo "Hello, $1"
echo "Your role is: $2"
```

Run:

```bash
./greet.sh Varun DevOps-Engineer
```

Output:

```text
Hello, Varun
Your role is: DevOps-Engineer
```

```text
$1 → Varun
$2 → DevOps-Engineer
```

---

## 5. `$#` — Number of Arguments

Create `count-args.sh`:

```bash
#!/bin/bash

echo "Number of arguments: $#"
```

Run:

```bash
./count-args.sh one two three
```

Output:

```text
Number of arguments: 3
```

### Real-World Example: Validate Input

```bash
#!/bin/bash

if [ $# -ne 2 ]; then
  echo "Usage: $0 <source-file> <destination-directory>"
  exit 1
fi

cp "$1" "$2"
```

Run:

```bash
./copy-file.sh notes.txt /tmp
```

This prevents the script from running with missing or incorrect inputs.

---

## 6. `$@` — All Arguments Separately

Create `show-args.sh`:

```bash
#!/bin/bash

for arg in "$@"; do
  echo "Argument: $arg"
done
```

Run:

```bash
./show-args.sh nginx docker kubernetes
```

Output:

```text
Argument: nginx
Argument: docker
Argument: kubernetes
```

### Important

Use:

```bash
"$@"
```

It keeps every argument separate, including arguments containing spaces.

Example:

```bash
./show-args.sh "Linux Fundamentals" "Shell Scripting"
```

Output:

```text
Argument: Linux Fundamentals
Argument: Shell Scripting
```

---

## 7. `$*` — All Arguments Together

```bash
#!/bin/bash

echo "$*"
```

Run:

```bash
./script.sh Linux Docker Kubernetes
```

Output:

```text
Linux Docker Kubernetes
```

### `$@` vs `$*`

```text
"$@" → Each argument stays separate
"$*" → All arguments become one combined value
```

For loops and scripts, prefer:

```bash
"$@"
```

---

## 8. `$?` — Exit Status

Every Linux command returns an exit status.

```text
0     → Success
Non-0 → Failure
```

Example:

```bash
#!/bin/bash

ls /etc
echo "Exit status: $?"
```

If `ls /etc` works:

```text
Exit status: 0
```

If a command fails:

```bash
ls /wrong-directory
echo "Exit status: $?"
```

Output:

```text
Exit status: 2
```

### Real-World Example: Check Whether a Service Is Running

```bash
#!/bin/bash

systemctl is-active --quiet nginx

if [ $? -eq 0 ]; then
  echo "Nginx is running"
else
  echo "Nginx is not running"
fi
```

A cleaner version:

```bash
#!/bin/bash

if systemctl is-active --quiet nginx; then
  echo "Nginx is running"
else
  echo "Nginx is not running"
fi
```

---

## 9. `$$` — Current Script Process ID

```bash
#!/bin/bash

echo "Current script PID: $$"
```

Output example:

```text
Current script PID: 24831
```

### Real-World Example: Prevent Two Copies of a Script

```bash
#!/bin/bash

LOCK_FILE="/tmp/backup-script.lock"

if [ -f "$LOCK_FILE" ]; then
  echo "Backup script is already running."
  exit 1
fi

echo "$$" > "$LOCK_FILE"

echo "Running backup..."
sleep 10

rm -f "$LOCK_FILE"
```

This can help prevent two backup jobs from running at the same time.

---

## 10. `$!` — Background Process ID

Use `&` to run a command in the background.

```bash
#!/bin/bash

sleep 60 &
PID=$!

echo "Background process PID: $PID"
```

```text
sleep 60 & → Runs in background
$!         → PID of that background process
```

### Real-World Example: Wait for a Background Job

```bash
#!/bin/bash

tar -czf backup.tar.gz /home/ec2-user/data &
BACKUP_PID=$!

echo "Backup started with PID: $BACKUP_PID"

wait "$BACKUP_PID"

echo "Backup job finished."
```

---

## 11. `$_` — Last Argument of Previous Command

```bash
mkdir project
cd $_
```

Here:

```text
$_ → project
```

So the second command becomes:

```bash
cd project
```

This is useful in the terminal, but use clear named variables in scripts when readability matters.

---

# Practical Examples

## 12. Backup Script

Create `backup.sh`:

```bash
#!/bin/bash

SOURCE_DIR="$1"
BACKUP_DIR="$2"

if [ $# -ne 2 ]; then
  echo "Usage: $0 <source-directory> <backup-directory>"
  exit 1
fi

if [ ! -d "$SOURCE_DIR" ]; then
  echo "Error: Source directory does not exist: $SOURCE_DIR"
  exit 1
fi

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_DIR/backup-$(date +%F).tar.gz" "$SOURCE_DIR"

if [ $? -eq 0 ]; then
  echo "Backup completed successfully."
else
  echo "Backup failed."
  exit 1
fi
```

Run:

```bash
./backup.sh /home/ec2-user/notes /home/ec2-user/backups
```

---

## 13. Check Disk Usage Script

Create `check-disk.sh`:

```bash
#!/bin/bash

THRESHOLD="${1:-80}"

USAGE=$(df / | awk 'NR==2 {gsub("%",""); print $5}')

echo "Current disk usage: ${USAGE}%"

if [ "$USAGE" -ge "$THRESHOLD" ]; then
  echo "Warning: Disk usage is above ${THRESHOLD}%"
  exit 1
else
  echo "Disk usage is healthy."
fi
```

Run with the default threshold:

```bash
./check-disk.sh
```

Run with a custom threshold:

```bash
./check-disk.sh 90
```

```text
$1          → Custom threshold
${1:-80}    → Use 80 when no argument is supplied
$?          → Can show whether the script succeeded or failed
```

---

## 14. Service Status Script

Create `service-status.sh`:

```bash
#!/bin/bash

SERVICE="$1"

if [ $# -ne 1 ]; then
  echo "Usage: $0 <service-name>"
  exit 1
fi

if systemctl is-active --quiet "$SERVICE"; then
  echo "$SERVICE is running."
else
  echo "$SERVICE is not running."
  exit 1
fi
```

Run:

```bash
./service-status.sh nginx
```

---

## 15. Process Multiple Log Files

Create `search-logs.sh`:

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
  echo "Usage: $0 <log-file-1> [log-file-2] ..."
  exit 1
fi

for LOG_FILE in "$@"; do
  echo "Checking: $LOG_FILE"
  grep -i "error" "$LOG_FILE"
done
```

Run:

```bash
./search-logs.sh app.log nginx.log system.log
```

```text
"$@" → Processes each log file separately
$#    → Checks whether files were supplied
$0    → Displays the script name in usage instructions
```

---

# Best Practices

- Start scripts with:

```bash
#!/bin/bash
```

- Quote variables:

```bash
"$1"
"$HOME"
"$FILE"
```

- Validate arguments before using them.
- Use meaningful variable names.
- Check command success using exit status.
- Use `"$@"` when looping through all arguments.
- Do not put passwords, API keys, private SSH keys, or secrets directly in scripts.
- Test scripts in a safe environment before using them in production.

---

# Quick Revision Cheat Sheet

```bash
$0      # Script name
$1      # First argument
$2      # Second argument
$#      # Number of arguments
"$@"    # All arguments, kept separately
"$*"    # All arguments as one value
$?      # Previous command exit status
$$      # Current script process ID
$!      # Last background process ID
$_      # Last argument of previous command
```

```text
Exit code 0     → Success
Exit code non-0 → Failure
```