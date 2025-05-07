Date: 05/07/2025

**1. Successful Logins**

Goal: Analyze successful SSH login attempts to identify patterns by user, source IP, and time. Assess whether any login behavior seems suspicious.
SPL Query:
```
index=main sourcetype=syslog "Accepted password"
rex "Accepted password for (?<user>\w+) from (?<src>\d+\.\d+\.\d+\.\d+)"
table _time user src
```




