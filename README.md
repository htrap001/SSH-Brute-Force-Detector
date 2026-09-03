# SSH Failed Login Detection Script

A Bash script designed to parse Linux authentication logs (`/var/log/auth.log` or custom mock logs) using standard Unix utilities (`awk`, `sort`, and `uniq`) to detect brute-force SSH attempts exceeding a predefined threshold.

---

## Bash Script (`detect_failed_ssh.sh`)

```bash
#!/usr/bin/env bash

LOG_FILE="${1:-/var/log/auth.log}"
THRESHOLD=5

fi

awk '/sshd.*Failed password/ {
    for (i = 1; i <= NF; i++) {
        if ($i == "from") {
            print $(i + 1)
            break
        }
    }
}' "$LOG_FILE" \
| sort \
| uniq -c \
| awk -v threshold="$THRESHOLD" '$1 > threshold {
    printf "ALERT: IP %s has %d failed attempts\n", $2, $1
}'
```
Give excecution permission to this script
```
chmod +x your_scripname.sh
```
Then with the help of AI i created a mock log file named mock.log
<img width="1920" height="1080" alt="Screenshot (588)" src="https://github.com/user-attachments/assets/869bba59-63f1-4280-b1ef-2060d61d2f6e" />
after this i ran my bash script
<img width="1920" height="1080" alt="Screenshot (586)" src="https://github.com/user-attachments/assets/876d824b-0b53-4601-a3f7-701d425edb9a" />
This shows the script is working fine.
---
