---
layout: post
title: Browsing and searching cloudwatch logs using the CLI
---

Test

```
# Find log groups
aws logs describe-log-groups --log-group-name-prefix staging/guardpost --output text

# Get a timestamp
date +%s

# Find all logs
aws logs filter-log-events --log-group-name staging/guardpost/app --start-time 1613572412000 --output text

# Filter logs
aws logs filter-log-events --log-group-name staging/guardpost/app --start-time 1613572412000 --output text --filter-pattern '5457187'
```
