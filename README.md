# SSH-Brute-Force-Detector
A Bash script that uses awk, sort, and uniq to parse /var/log/auth.log and identify IPs with more than 5 failed SSH attempts.
#!/usr/bin/env bash

LOG_FILE="${1:-/var/log/auth.log}"
THRESHOLD=5

if [[ ! -r "$LOG_FILE" ]]; then
  echo "Error: Cannot read log file '$LOG_FILE'" >&2
  exit 1
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
