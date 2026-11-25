# Assignment 1: Linux Operating System --- Questions and Answers

**Module:** 2 --- Hands-on Assignment\
**Scenario:** Meridian Retail reporting server (`rpt-01`)\
**Duration:** 90--120 minutes\
**Total:** 100 marks

> **Important:** This document follows the uploaded assignment. Commands
> and explanations are provided as a guide. Replace every
> `[REPLACE WITH YOUR OUTPUT]` or `TODO` with results from your own
> Linux machine. Do not invent terminal output. The assignment allows
> Ubuntu 22.04 or Rocky Linux 9 and requires sudo access.

## Scenario and objectives

You have joined the Platform Engineering team at Meridian Retail. The
`rpt-01` server generates nightly sales reports. The previous
administrator left limited documentation, and Finance reports that the
previous night's report did not arrive.

Your work is to: 1. Inventory the Linux machine. 2. Create a restricted
account and configure safe group permissions. 3. Configure SSH key
authentication and practise secure file transfer. 4. Investigate a
simulated reporting failure using logs and service troubleshooting.

## Before starting

Create the working directory and answer file:

``` bash
mkdir -p ~/assignment1
cd ~/assignment1
touch answers.md
```

Start a terminal transcript:

``` bash
script ~/assignment1/transcript.log
```

Keep all assignment files under `~/assignment1`. Use the command line
only. If a command fails, read the error message before changing
anything.

**Safety rules from the assignment** - Never run `rm -rf` with a
wildcard. - Do not use `chmod 777` to solve permission problems. - Never
copy or upload your private SSH key. - Do not apply SSH daemon hardening
changes during this task; only identify the required directives. - If
SSH configuration is ever changed in a separate exercise, keep a second
working session open and verify access before closing the first.

------------------------------------------------------------------------

# Task 1 --- Learn your way around the server

**Orientation and inventory \| Easy \| 20 minutes \| 15 marks**

### Question 1. Record the hostname, kernel version, and uptime.

**Commands:**

``` bash
hostname
uname -r
uptime
```

Alternative detailed kernel/OS information:

``` bash
hostnamectl
uname -a
```

**Explanation:** - `hostname` prints the system hostname. - `uname -r`
prints the running kernel release. - `uptime` shows how long the system
has been running, plus load averages.

**Answer to record in `answers.md`:**

``` text
1. Hostname: [REPLACE WITH YOUR OUTPUT]
   Kernel version: [REPLACE WITH YOUR OUTPUT]
   Uptime: [REPLACE WITH YOUR OUTPUT]
   Commands used: hostname; uname -r; uptime
```

### Question 2. Report total, used, and available disk space for every mounted filesystem in human-readable units.

**Command:**

``` bash
df -hT
```

If you want to omit temporary filesystems, inspect the output carefully
and explain what you excluded; the assignment asks for every mounted
filesystem.

**Explanation:** - `df` reports filesystem space usage. - `-h` uses
human-readable units such as K, M, and G. - `-T` also displays
filesystem type.

**Answer to record:**

``` text
2. Disk usage:
   [PASTE RELEVANT df -hT OUTPUT]
   Command used: df -hT
```

### Question 3. Identify the five largest directories under `/var` and record their sizes.

**Command:**

``` bash
sudo du -x -h --max-depth=1 /var 2>/dev/null | sort -h -r | head -n 5
```

**Explanation:** - `du` estimates disk usage. - `-x` stays on the same
filesystem. - `-h` displays readable units. - `--max-depth=1` reports
immediate entries under `/var`. - `sort -h -r` sorts human-readable
sizes from largest to smallest. - `head -n 5` prints the first five
results.

**Answer to record:**

``` text
3. Five largest directories under /var:
   [PASTE YOUR FIVE RESULTS]
   Command used: sudo du -x -h --max-depth=1 /var 2>/dev/null | sort -h -r | head -n 5
```

### Question 4. List `/etc` in long format with human-readable sizes and count its entries.

**Commands:**

``` bash
ls -lh /etc
ls -1A /etc | wc -l
```

**Explanation:** - `ls -lh` shows a long listing and human-readable file
sizes. - `ls -1A` lists one entry per line and includes hidden entries
other than `.` and `..`. - `wc -l` counts the entries listed.

**Answer to record:**

``` text
4. /etc listing: [PASTE OR SUMMARIZE OUTPUT]
   Number of entries: [REPLACE WITH COUNT]
   Commands used: ls -lh /etc; ls -1A /etc | wc -l
```

### Question 5. Find every `.conf` file under `/etc` modified in the last 30 days and record the count.

**Commands:**

``` bash
sudo find /etc -type f -name '*.conf' -mtime -30 -print
sudo find /etc -type f -name '*.conf' -mtime -30 -print | wc -l
```

**Explanation:** - `find /etc` searches recursively under `/etc`. -
`-type f` selects regular files. - `-name '*.conf'` selects names ending
in `.conf`. - `-mtime -30` selects files modified less than 30 periods
of 24 hours ago. - `wc -l` counts the matching paths.

**Answer to record:**

``` text
5. Matching .conf files:
   [PASTE MATCHING PATHS]
   Count: [REPLACE WITH COUNT]
   Commands used: sudo find /etc -type f -name '*.conf' -mtime -30 -print
                  sudo find /etc -type f -name '*.conf' -mtime -30 -print | wc -l
```

### Question 6. Show the ten processes using the most CPU and record the top three.

**Command:**

``` bash
ps -eo pid,ppid,comm,%cpu,%mem --sort=-%cpu | head -n 11
```

**Explanation:** - `ps -e` selects all processes. - `-o` selects
displayed columns. - `--sort=-%cpu` sorts highest CPU usage first. -
`head -n 11` includes the header plus ten processes.

**Answer to record:**

``` text
6. Top ten CPU-consuming processes:
   [PASTE OUTPUT]
   Top three: [RECORD THE TOP THREE PROCESSES]
   Command used: ps -eo pid,ppid,comm,%cpu,%mem --sort=-%cpu | head -n 11
```

### Task 1 verification

-   [ ] `answers.md` contains six numbered answers.
-   [ ] Each answer includes the command used.
-   [ ] Disk and directory sizes use human-readable units.
-   [ ] Actual output is recorded, not guessed.

------------------------------------------------------------------------

# Task 2 --- Set up a colleague safely

**Users, groups and permissions \| Moderate \| 25 minutes \| 25 marks**

Priya needs read/write access to report output, but must not be able to
change report configuration and must not have administrative rights.

### Question 1. Create group `reporting` and user `priya` with a home directory, Bash login shell, and password.

**Commands:**

``` bash
sudo groupadd reporting
sudo useradd -m -s /bin/bash priya
sudo passwd priya
```

**Explanation:** - `groupadd` creates the group. - `useradd -m` creates
the user's home directory. - `-s /bin/bash` sets Bash as the login
shell. - `passwd` interactively sets the account password.

If the group or user already exists, inspect it first rather than
blindly recreating it:

``` bash
getent group reporting
id priya
```

**Answer to record:**

``` text
1. Group created: reporting
   User created: priya
   Home directory: [VERIFY WITH getent passwd priya OR ls -ld /home/priya]
   Login shell: /bin/bash
   Password set: [YES/NO]
   Commands used: [RECORD COMMANDS AND ACTUAL RESULT]
```

### Question 2. Add Priya to `reporting`, prove membership, and record how you proved it.

**Commands:**

``` bash
sudo usermod -aG reporting priya
id priya
getent group reporting
```

**Explanation:** - `usermod -aG` appends the supplementary group without
removing existing supplementary group memberships. - `id priya` displays
the user's UID, primary GID, and supplementary groups. -
`getent group reporting` displays the group record and its listed
members.

A new login session may be required for the user's active session to
receive the updated group list.

**Answer to record:**

``` text
2. Priya added to reporting.
   Proof: [PASTE id priya OUTPUT]
   Command used: sudo usermod -aG reporting priya
```

### Question 3. Create `/opt/reports`, owned by root with group ownership `reporting`.

**Commands:**

``` bash
sudo mkdir -p /opt/reports
sudo chown root:reporting /opt/reports
ls -ld /opt/reports
```

**Explanation:** `chown root:reporting` sets the owner to `root` and the
group owner to `reporting`.

**Answer to record:**

``` text
3. Directory: /opt/reports
   Owner: root
   Group: reporting
   Verification: [PASTE ls -ld OUTPUT]
   Commands used: sudo mkdir -p /opt/reports; sudo chown root:reporting /opt/reports
```

### Question 4. Give the reporting group read, write, and directory-enter permissions; give everyone else no permissions. Record the numeric mode and explain it.

**Command:**

``` bash
sudo chmod 2770 /opt/reports
ls -ld /opt/reports
```

**Explanation:** - Owner permissions `7` = read, write, execute. - Group
permissions `7` = read, write, execute. - Other permissions `0` = no
access. - Leading `2` sets the setgid bit on the directory.

The setgid bit causes new files and subdirectories created inside this
directory to inherit the directory's group (`reporting`) on typical
Linux filesystems.

**Numeric mode:** `2770`.

**Answer to record:**

``` text
4. Permission mode: 2770
   Explanation: root and reporting have full directory access, others have no access, and setgid makes new entries inherit the reporting group.
   Verification: [PASTE ls -ld /opt/reports OUTPUT]
   Command used: sudo chmod 2770 /opt/reports
```

### Question 5. Ensure files created inside `/opt/reports` automatically belong to `reporting`.

**Command / verification:**

``` bash
ls -ld /opt/reports
```

The `s` in the group-execute position (for example, `drwxrws---`)
indicates setgid is enabled. It was enabled by mode `2770` in Question
4.

You can test group inheritance after switching to Priya and creating a
test file in Question 7:

``` bash
ls -l /opt/reports
```

**Answer to record:**

``` text
5. Group inheritance: setgid enabled on /opt/reports.
   Verification: [PASTE ls -ld OUTPUT AND, IF TESTED, NEW FILE GROUP]
   Command used: sudo chmod 2770 /opt/reports
```

### Question 6. Create `/opt/reports/config.ini` owned by root and readable by reporting, but not modifiable by the group.

**Commands:**

``` bash
sudo touch /opt/reports/config.ini
sudo chown root:reporting /opt/reports/config.ini
sudo chmod 640 /opt/reports/config.ini
ls -l /opt/reports/config.ini
```

**Explanation:** - Mode `640` means owner `rw-`, group `r--`, others
`---`. - Root can read and modify the file. - Members of `reporting` can
read but cannot write it. - Others have no access.

**Answer to record:**

``` text
6. File: /opt/reports/config.ini
   Owner: root
   Group: reporting
   Mode: 640
   Verification: [PASTE ls -l OUTPUT]
   Commands used: sudo touch ...; sudo chown root:reporting ...; sudo chmod 640 ...
```

### Question 7. Switch to Priya and attempt to create a report file, read `config.ini`, and modify `config.ini`.

First start a fresh login session for Priya:

``` bash
su - priya
```

Attempt to create a file:

``` bash
touch /opt/reports/priya-test.txt
```

Read the configuration:

``` bash
cat /opt/reports/config.ini
```

Attempt to modify it:

``` bash
echo '# test change' >> /opt/reports/config.ini
```

Inspect the result (return to an account with permission if necessary):

``` bash
ls -l /opt/reports
```

**Expected permission behaviour if ownership and modes are configured
correctly:**

  -----------------------------------------------------------------------
  Attempt                 Expected result         Reason
  ----------------------- ----------------------- -----------------------
  Create a file in        Succeeds                Priya is in
  `/opt/reports`                                  `reporting`, and the
                                                  directory grants group
                                                  `rwx`

  Read `config.ini`       Succeeds                The file grants group
                                                  read permission

  Modify `config.ini`     Fails with permission   The file grants the
                          denied                  group read-only
                                                  permission
  -----------------------------------------------------------------------

**Important:** These are expected results, not proof of what happened on
your machine. Record your actual results and errors.

**Answer to record:**

``` text
7. Tested as user priya:
   Create file: [SUCCESS/FAILURE + ACTUAL OUTPUT]
   Read config.ini: [SUCCESS/FAILURE + ACTUAL OUTPUT]
   Modify config.ini: [SUCCESS/FAILURE + ACTUAL OUTPUT]
   Commands used: su - priya; touch ...; cat ...; echo ... >> ...
```

### Task 2 verification

Run from an account with permission to inspect the directory:

``` bash
ls -ld /opt/reports
ls -l /opt/reports/config.ini
id priya
```

Paste all three outputs into your evidence section.

### Think about it: If Priya receives `Permission denied` when creating a file, what three things would you check, in order?

**Answer:**

1.  **Directory ownership and permissions:** Check whether
    `/opt/reports` is owned by `root:reporting` and has group write and
    execute permissions.

    ``` bash
    ls -ld /opt/reports
    ```

2.  **Priya's group membership:** Check whether `reporting` appears in
    her group list.

    ``` bash
    id priya
    ```

3.  **Active login session and path/access controls:** Group membership
    changes require a fresh login session; also confirm the path and
    inspect any ACLs if ordinary ownership/mode bits look correct.

    ``` bash
    su - priya
    getfacl /opt/reports
    ```

------------------------------------------------------------------------

# Task 3 --- Remote access without passwords

**SSH, key authentication and file transfer \| Moderate \| 25 minutes \|
25 marks**

### Question 1. Generate an SSH key pair using Ed25519 and a comment identifying the machine.

**Commands:**

``` bash
ssh-keygen -t ed25519 -C "$(hostname)-assignment1"
```

Press Enter to accept the default key location, normally
`~/.ssh/id_ed25519`. Set a passphrase if appropriate for your lab.

Verify the files:

``` bash
ls -l ~/.ssh
```

**Explanation:** Ed25519 is a modern public-key cryptography algorithm
supported by OpenSSH. The command creates a private key and a matching
public key.

**Answer to record:**

``` text
1. Key algorithm: ed25519
   Comment: [REPLACE WITH hostname]
   Key location: ~/.ssh/id_ed25519
   Public key: ~/.ssh/id_ed25519.pub
   Command used: ssh-keygen -t ed25519 -C "$(hostname)-assignment1"
```

### Question 2. Which generated file may be copied to a server, and which must never leave your machine?

**Answer:** The public key (`id_ed25519.pub`) may be installed on the
server; the private key (`id_ed25519`) must remain secret on the client
machine and must never be copied to the server or shared.

### Question 3. Install the public key so you can log in without a password. State the exact file it ends up in.

If the remote account and server are available and password-based
initial access is permitted:

``` bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@server-address
```

The public key is appended to:

``` text
~/.ssh/authorized_keys
```

For a local lab, the public key can be installed into the target
account's `~/.ssh/authorized_keys` file, ensuring the directory and file
permissions below are correct.

**Answer to record:**

``` text
3. Public key installed in: ~/.ssh/authorized_keys (on the target account)
   Installation command: [RECORD THE COMMAND YOU ACTUALLY USED]
   Login test: [RECORD ACTUAL RESULT]
```

### Question 4. Set the permissions on `~/.ssh` and the private key to values required by SSH.

**Commands:**

``` bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
ls -ld ~/.ssh
ls -l ~/.ssh
```

**Explanation:** - `~/.ssh` mode `700`: only the owner can access the
directory. - Private key mode `600`: only the owner can read and write
it. - Public key mode `644` is commonly acceptable because it is not
secret.

The assignment specifically asks you to record the `.ssh` directory and
private-key permissions.

**Answer to record:**

``` text
4. ~/.ssh permissions: 700
   Private key permissions: 600
   Public key permissions: 644
   Verification: [PASTE ls OUTPUT]
```

### Question 5. Deliberately set the private key to mode 644, attempt to connect, record SSH's exact message, and restore the correct mode.

**Commands:**

``` bash
chmod 644 ~/.ssh/id_ed25519
ssh -i ~/.ssh/id_ed25519 username@server-address
```

Record the exact warning/error from your SSH client. OpenSSH commonly
rejects a private key with permissions that are too open and may report
that it is "UNPROTECTED PRIVATE KEY FILE" or that it "Load key ...: bad
permissions"; the exact wording varies by version and platform.

Restore the secure mode immediately:

``` bash
chmod 600 ~/.ssh/id_ed25519
```

**Answer to record:**

``` text
5. Deliberately insecure mode: 644
   Exact SSH message from my machine:
   "[PASTE THE EXACT MESSAGE HERE]"
   Restored mode: 600
   Command used: chmod 644 ~/.ssh/id_ed25519; [SSH COMMAND]; chmod 600 ~/.ssh/id_ed25519
```

Do not claim a specific error message unless you actually reproduced it.

### Question 6. Copy `/opt/reports/config.ini` to your machine using `scp`, then copy it back into your home directory under a new name.

Run these commands from the client machine. Replace the username and
host with your actual values.

Copy from remote server to the current local directory:

``` bash
scp username@server-address:/opt/reports/config.ini ./config.ini
```

Copy it back to the remote user's home directory under a new name:

``` bash
scp ./config.ini username@server-address:~/config-backup.ini
```

Verify the remote copy:

``` bash
ssh username@server-address 'ls -l ~/config-backup.ini'
```

**Answer to record:**

``` text
6. Downloaded file: [LOCAL PATH]
   Uploaded copy: ~/config-backup.ini on remote host
   Verification: [PASTE ACTUAL OUTPUT]
   Commands used: [RECORD THE SCP COMMANDS]
```

If the lab is entirely local and does not have a separate SSH
server/client, clearly state how you simulated or could not perform the
remote transfer. Do not pretend an SCP transfer happened.

### Question 7. Identify the `sshd_config` settings that disable password authentication and prevent direct root login. Do not apply them.

Inspect the configuration:

``` bash
sudo grep -E '^[# ]*(PasswordAuthentication|PermitRootLogin)' /etc/ssh/sshd_config
```

The directives and desired values are:

``` text
PasswordAuthentication no
PermitRootLogin no
```

**Explanation:** - `PasswordAuthentication no` disables password-based
SSH authentication. - `PermitRootLogin no` prevents direct SSH login as
root.

The assignment says to identify these settings only. Do not edit or
apply them for this task.

**Answer to record:**

``` text
7. PasswordAuthentication no
   PermitRootLogin no
   Inspection command: sudo grep -E '^[# ]*(PasswordAuthentication|PermitRootLogin)' /etc/ssh/sshd_config
   Changes applied: No (as instructed)
```

### Task 3 verification

``` bash
ls -l ~/.ssh
```

Include: - \[ \] Output showing `.ssh` directory and key files. - \[ \]
Exact SSH error message from the mode `644` test. - \[ \] The two SSH
configuration directives and their desired values. - \[ \] No private
key content in `answers.md`, screenshots, or GitHub.

------------------------------------------------------------------------

# Task 4 --- Investigate the failed report

**Incident investigation \| Complex \| 35 minutes \| 25 marks**

Finance did not receive last night's report. The service is called
`reporting.service`. The assignment asks you to create a simulated log
first, analyse it, extract errors, inspect the service, and write an
incident summary.

## Set up the simulated log

Run as a normal user:

``` bash
mkdir -p ~/assignment1/lab/logs
cd ~/assignment1/lab

for i in $(seq 1 500); do
  echo "$(date +%F' '%T) INFO processing record $i" >> logs/report.log
done

echo "$(date +%F' '%T) ERROR disk quota exceeded" >> logs/report.log
echo "$(date +%F' '%T) ERROR report generation aborted" >> logs/report.log

for i in $(seq 1 40); do
  echo "$(date +%F' '%T) INFO retry scheduled" >> logs/report.log
done
```

This creates 500 INFO processing lines, 2 ERROR lines, and 40 retry INFO
lines.

### Question 1. Count total log lines and lines containing `ERROR`.

**Commands:**

``` bash
wc -l logs/report.log
grep -c 'ERROR' logs/report.log
```

**Expected counts for the exact supplied setup:** 542 total lines and 2
lines containing `ERROR`, assuming the commands ran once and completed
successfully.

**Answer to record:**

``` text
1. Total log lines: [VERIFY WITH wc -l]
   ERROR lines: [VERIFY WITH grep -c]
   Commands used: wc -l logs/report.log; grep -c 'ERROR' logs/report.log
```

### Question 2. Print only ERROR lines, record the first one, and explain why it matters more than the last.

**Commands:**

``` bash
grep 'ERROR' logs/report.log
grep 'ERROR' logs/report.log | head -n 1
```

**Explanation:** The first error is often the initiating failure. Later
errors may be consequences of the original failure. In this supplied
scenario, `disk quota exceeded` appears before
`report generation aborted`, suggesting the quota problem may have
caused report generation to stop.

**Answer to record:**

``` text
2. ERROR lines:
   [PASTE BOTH ERROR LINES]
   First error: [PASTE FIRST ERROR LINE]
   Explanation: The first error is the likely root cause; the later report-aborted error may be a consequence.
   Commands used: grep 'ERROR' logs/report.log; grep 'ERROR' logs/report.log | head -n 1
```

### Question 3. Show the final 20 lines without opening an editor.

**Command:**

``` bash
tail -n 20 logs/report.log
```

**Answer to record:**

``` text
3. Final 20 lines:
   [PASTE OUTPUT]
   Command used: tail -n 20 logs/report.log
```

### Question 4. Using a single pipeline, count lines containing the word `retry`.

**Command:**

``` bash
grep -w 'retry' logs/report.log | wc -l
```

**Expected count for the supplied setup:** `40`.

**Explanation:** `grep -w` matches the whole word `retry`; `wc -l`
counts matching lines.

**Answer to record:**

``` text
4. Retry line count: [VERIFY OUTPUT; expected 40]
   Command used: grep -w 'retry' logs/report.log | wc -l
```

### Question 5. Write ERROR lines to `~/assignment1/lab/errors.txt` and confirm the file contains exactly what you expect.

**Commands:**

``` bash
grep 'ERROR' logs/report.log > ~/assignment1/lab/errors.txt
cat ~/assignment1/lab/errors.txt
wc -l ~/assignment1/lab/errors.txt
```

**Explanation:** `>` redirects the matching ERROR lines into a new file,
replacing its contents if it already exists.

**Expected:** Two lines, matching the two ERROR lines in `report.log`.

**Answer to record:**

``` text
5. errors.txt contents:
   [PASTE OUTPUT]
   Number of lines: [VERIFY; expected 2]
   Commands used: grep 'ERROR' logs/report.log > ~/assignment1/lab/errors.txt
                  cat ~/assignment1/lab/errors.txt
                  wc -l ~/assignment1/lab/errors.txt
```

### Question 6. Check whether `reporting.service` exists. If it does not, name commands to check whether a service is running, start it, and read its logs.

**Commands:**

``` bash
systemctl status reporting.service
systemctl is-active reporting.service
```

If the service exists but is stopped, starting it would be:

``` bash
sudo systemctl start reporting.service
```

Read its logs:

``` bash
sudo journalctl -u reporting.service
```

For recent logs:

``` bash
sudo journalctl -u reporting.service -n 50 --no-pager
```

**Explanation:** `systemctl status` shows whether systemd knows the unit
and its current state; `systemctl start` requests a start;
`journalctl -u` reads logs for that unit.

The assignment says if the service does not exist, explain the three
commands. Do not create a fake service just to claim it exists.

**Answer to record:**

``` text
6. Service check result: [EXISTS / UNIT NOT FOUND / OTHER ACTUAL RESULT]
   Check command: systemctl status reporting.service
   Start command (if unit exists): sudo systemctl start reporting.service
   Log command: sudo journalctl -u reporting.service
   If missing: Explain that this lab machine does not have the supplied service unit.
```

### Question 7. Give five troubleshooting checks, in order, for a service that fails overnight, with a command for each.

**Suggested sequence:**

1.  **Check service state and recent failure details.**

    ``` bash
    systemctl status reporting.service --no-pager
    ```

2.  **Read service logs around the failure.**

    ``` bash
    sudo journalctl -u reporting.service --since "yesterday" --no-pager
    ```

3.  **Check disk space and inodes**, because the simulated first error
    is `disk quota exceeded`.

    ``` bash
    df -h
    df -i
    ```

4.  **Check permissions and ownership** on the report output and
    configuration paths.

    ``` bash
    ls -ld /opt/reports
    ls -l /opt/reports
    ```

5.  **Check scheduling and dependencies**, such as the timer/cron entry
    and required mounts or network dependencies.

    ``` bash
    systemctl list-timers --all
    systemctl list-dependencies reporting.service
    ```

**Explanation:** Start with the unit's state and logs, then validate
common resource and access problems, then investigate scheduling and
dependencies. The exact order may be adapted to evidence from the
incident; document why each check is next.

**Answer to record:**

``` text
7. Five checks in order:
   1. Service state: systemctl status reporting.service --no-pager
   2. Service logs: sudo journalctl -u reporting.service --since "yesterday" --no-pager
   3. Disk space/inodes: df -h; df -i
   4. Ownership and permissions: ls -ld /opt/reports; ls -l /opt/reports
   5. Scheduling/dependencies: systemctl list-timers --all; systemctl list-dependencies reporting.service
```

### Question 8. Open `answers.md` in `vi`, add an incident summary, and save and exit from within `vi`.

Open the file:

``` bash
vi ~/assignment1/answers.md
```

In `vi`: 1. Press `G` to move to the end of the file. 2. Press `o` to
open a new line below and enter insert mode. 3. Add the incident
summary. 4. Press `Esc`. 5. Type `:wq` and press Enter to save and quit.

**Suggested incident summary (edit it to reflect your actual
findings):**

``` text
Incident summary:
The simulated reporting log contains 542 lines: 500 processing INFO entries, two ERROR entries, and 40 retry INFO entries. The first error is "disk quota exceeded"; the next error says report generation aborted. The first error is the likely initiating problem, while the abort message is a downstream symptom. I identified the issue by counting and filtering the log with wc and grep, reviewing the final lines with tail, and extracting the ERROR entries into errors.txt. To reduce recurrence, I would monitor filesystem and quota usage, alert before the quota is exhausted, rotate and retain logs appropriately, and validate the reporting service and its schedule after changes.
```

**Important:** This summary describes the supplied simulated log. Add
the actual `reporting.service` check result from your machine before
submission.

**Answer to record:**

``` text
8. Incident summary added to answers.md using vi.
   Save/exit command: :wq
   Verification: [REOPEN WITH vi OR DISPLAY USING cat/less AND CONFIRM CONTENT]
```

### Task 4 verification

``` bash
ls -l ~/assignment1/lab/errors.txt
cat ~/assignment1/lab/errors.txt
wc -l ~/assignment1/lab/errors.txt
```

Confirm that: - \[ \] `errors.txt` exists. - \[ \] It contains only the
intended ERROR lines. - \[ \] `answers.md` contains the incident
summary. - \[ \] The incident summary was added and saved using `vi`. -
\[ \] The five troubleshooting checks include a command each.

------------------------------------------------------------------------

# Final submission checklist

The assignment requires one archive named
`assignment1-<yourname>.tar.gz` containing:

-   `answers.md` --- an answer and command for every numbered step.
-   `transcript.log` --- terminal transcript.
-   `lab/errors.txt` --- extracted ERROR lines.
-   Screenshot or pasted output of `ls -ld /opt/reports` and `id priya`.

Before packaging, stop the `script` transcript by typing:

``` bash
exit
```

Check the required files:

``` bash
cd ~/assignment1
ls -l
ls -l lab/errors.txt
```

Create the archive. Replace `varunPandian` if you want a different
submission name:

``` bash
tar -czvf assignment1-varunPandian.tar.gz -C ~ assignment1
```

Verify the archive contents:

``` bash
tar -tzvf assignment1-varunPandian.tar.gz
```

The archive should contain the `assignment1/` directory with
`answers.md`, `transcript.log`, and `lab/errors.txt`.

**Before submitting:** - \[ \] Every numbered question has an answer. -
\[ \] Every answer records the command used. - \[ \] Machine-specific
outputs are real outputs from your own system. - \[ \] The Priya account
and permission tests were performed and documented. - \[ \] SSH key
permissions and the deliberately reproduced SSH error are documented. -
\[ \] No private key, password, token, or secret is included. - \[ \]
`errors.txt` contains the intended two error lines for the supplied
scenario. - \[ \] The incident summary was saved using `vi`. - \[ \]
Archive contents were verified with `tar -tzvf`.

## Marking rubric from the assignment

  ------------------------------------------------------------------------
  Criterion                                    Marks Evidence
  --------------------- ---------------------------- ---------------------
  Task 1 ---                                      15 Six answers, correct
  orientation and                                    commands,
  inventory                                          human-readable units

  Task 2 --- users,                               25 Correct ownership,
  groups, permissions                                mode, group
                                                     inheritance, tested
                                                     as the user

  Task 3 --- SSH and                              25 Key pair, correct
  key authentication                                 permissions, error
                                                     reproduced, hardening
                                                     identified

  Task 4 ---                                      25 Log analysis correct,
  investigation                                      errors extracted,
                                                     sequence sound, `vi`
                                                     used

  Quality of write-up                             10 Clear, accurate,
                                                     reproducible by
                                                     someone else

  **Total**                                  **100** 
  ------------------------------------------------------------------------

## Common pitfalls from the assignment

-   A group change takes effect in a new login session; test with a
    fresh session.
-   Do not use `chmod 777` to make permission problems disappear.
-   Investigate the first error, not just the last one; later failures
    may be consequences.
-   Verify actual files and permissions with `ls -l`; do not assume a
    file exists or has the expected access.

------------------------------------------------------------------------

## Personal completion tracker

Mark a task complete only after performing it and saving its real
evidence.

-   [ ] Task 1 --- Learn your way around the server
-   [ ] Task 2 --- Set up Priya safely
-   [ ] Task 3 --- SSH and file transfer
-   [ ] Task 4 --- Investigate the failed report
-   [ ] Final archive created and verified
