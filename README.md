# SSH Failed Login Detection Script

A Bash script designed to parse Linux authentication logs (`/var/log/auth.log` or custom mock logs) using standard Unix utilities (`awk`, `sort`, and `uniq`) to detect brute-force SSH attempts exceeding a predefined threshold.

---

## Bash Script (`detect_failed_ssh.sh`)

```bash
#!/usr/bin/env bash

# Default log file path or accept from first argument
LOG_FILE="${1:-/var/log/auth.log}"
THRESHOLD=5

# Verify log file exists and is readable
if [[ ! -r "$LOG_FILE" ]]; then
  echo "Error: Cannot read log file '$LOG_FILE'" >&2
  exit 1
fi

# Pipeline:
# 1. awk: Filter for failed password lines and extract the IP address following 'from'
# 2. sort: Sort IPs lexicographically so duplicate entries are adjacent
# 3. uniq -c: Aggregate adjacent duplicates and prefix with occurrence count
# 4. awk: Filter IPs with count > THRESHOLD and print alert message
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

---
