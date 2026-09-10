Shell scripting is important in DevOps because it allows us to automate repetitive Linux, server, deployment, cloud, Docker, Kubernetes, 
and CI/CD tasks, reducing manual effort and errors.

# Shell Scripting

> Shell scripting is the practice of combining Linux commands into reusable scripts to automate repetitive operational tasks.

## Why Shell Scripting?

Shell scripts help automate tasks such as:

- System health checks
- Log analysis
- File and directory operations
- Backups
- Service management
- Deployment and operational tasks
- CI/CD automation

---

## 1. Basic Shell Script

```bash
#!/bin/bash

echo "Hello, $USER"
echo "Today is $(date +%F)" 
```

## 2. Variables
   Variables store values that can be reused in a script.
   name="DevOps"
   count=5

name="DevOps"
count=5

echo "$name"
echo "$count"

Bash variable assignments do not contain spaces around =.

Command Substitution
Store command output in a variable:
today=$(date +%F)
echo "$today"

## 3. Script Arguments
   Arguments allow input to be passed when executing a script.
   ./backup.sh /opt/app
   Here /opt/app is the first argument.
   Inside the script:
   echo "Backup source: $1"
   Common special variables:
   Variable	Meaning
   $0	Script name
   $1	First argument
   $2	Second argument
   $#	Number of arguments
   $?	Exit code of previous command


Example:
./backup.sh /opt/app /backup
$1 → /opt/app
$2 → /backup
$# → 2

## 4. Exit Codes
   Every command returns an exit status.
   0     → Success
   non-0 → Failure
   Check the previous command:
   echo $?
   Scripts can explicitly return a status:
   exit 0
   or:
   exit 1
   DevOps Importance
   Exit codes are important in:
- CI/CD pipelines
- Cron jobs
- Monitoring
- Automation
  A non-zero exit code can indicate that an automated task failed.

## 5. Conditionals
   Use if to make decisions.
   if [ -f "$1" ]; then
   echo "File exists"
   elif [ -d "$1" ]; then
   echo "Directory exists"
   else
   echo "Path not found" >&2
   exit 1
   fi
   Common tests:
   -f → file exists
   -d → directory exists
   -z → string is empty
   -n → string is not empty
   -eq → equal
   -ne → not equal
   -gt → greater than
   -lt → less than

## 6. Loops
   for Loop
   Used to repeat an operation over a list of values.
   for file in /var/log/*.log; do
   echo "$file"
   done
   while Loop
   Runs while a condition remains true.
   count=1

while [ "$count" -le 3 ]; do
echo "Attempt $count"
count=$((count + 1))
done
break and continue
break    → exit the loop
continue → skip the current iteration

## 7. Functions
   Functions group reusable logic.
   log() {
   echo "[$(date +%T)] $1"
   }

log "Starting backup"
Benefits:
- Reusable logic
- Cleaner scripts
- Easier maintenance
- Less duplicated code
  Use local for function-specific variables:
  backup() {
  local source="$1"
  }

## 8. User Input
   Read input from the terminal:
   read -p "Username: " user
   echo "Welcome, $user"
   For hidden input:
   read -sp "Password: " password

## 9. Standard Output & Error
   Linux has separate output streams:
   stdout (1) → normal output
   stderr (2) → error output
   Send an error to stderr:
   echo "Backup failed" >&2
   Redirect output:
   command > output.txt
   Redirect errors:
   command 2> error.txt
   Append output:
   command >> output.txt

## 10. Reliable Shell Scripts
    A useful Bash safety practice:
    set -euo pipefail
    Meaning:
    -e → stop when a command fails
    -u → detect undefined variables
    pipefail → detect failures inside pipelines
    This helps prevent scripts from silently continuing after failures.

## 11. Shell Scripting Best Practices
    Validate input
    if [ $# -lt 1 ]; then
    echo "Usage: $0 <path>" >&2
    exit 1
    fi
    Quote variables
    Prefer:
    "$file"
    instead of:
    $file
    Quoting helps handle paths containing spaces safely.
    Avoid hardcoded credentials
    Do not store passwords, API keys or secrets directly inside scripts.
    Use environment variables or a proper secrets-management system.
    Be careful with destructive commands
    Avoid unsafe use of:
    rm -rf
    Always validate paths and inputs before performing destructive operations.
    Use ShellCheck
    Run ShellCheck against shell scripts to identify common scripting mistakes and potential problems.
## 12. Practical DevOps Example
    A health-check script can combine:
    Input
    ↓
    Validation
    ↓
    Disk Check
    ↓
    Memory Check
    ↓
    Uptime Check
    ↓
    Logging
    ↓
    Exit Code
    Example:
    #!/bin/bash

## set -euo pipefail

echo "===== System Health Check ====="
echo "Time: $(date)"

echo "Disk:"
df -h

echo "Memory:"
free -h

echo "Uptime:"
uptime
This demonstrates how Linux commands can be combined into reusable operational automation.
## Key Takeaways
Shell Scripting
↓
Linux Commands
↓
Variables + Arguments
↓
Conditions + Loops + Functions
↓
Error Handling + Exit Codes
↓
Reliable Automation
## Core commands/concepts learned
#!/bin/bash
chmod +x
./script.sh
variables
$1 / $2 / $#
$?
if / elif / else
for / while
break / continue
functions
read
stdout / stderr
> / >> / 2>
set -euo pipefail
ShellCheck
## DevOps Relevance
Shell scripting provides a foundation for:
- Linux administration
- CI/CD automation
- Deployment automation
- Monitoring and health checks
- Backup automation
- Troubleshooting
- Cloud/server operations