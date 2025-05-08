Date: 05/07/2025

**1. Successful Logins**

Goal: Analyze successful SSH login attempts to identify patterns by user, source IP, and time. Assess whether any login behavior seems suspicious.
SPL Query:
```
index=main sourcetype=syslog "Accepted password"
rex "Accepted password for (?<user>\w+) from (?<src>\d+\.\d+\.\d+\.\d+)"
table _time user src
```

Note: Several successful SSH logins were seen from external IPs, such as 8.8.8.8 and 203.0.113.12. These IPs are not associated with internal traffic and accessed sensitive accounts like root, guest, and john. While no high-frequency signs of brute force was detected, the repeated logins from foreign IPs over multiple days might be the credential compromise or unauthorized access. Further investigation is recommended to determine if these accounts might be misused.


 



