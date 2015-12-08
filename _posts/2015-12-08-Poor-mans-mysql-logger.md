---
title: "Poor man's MySQL logger"
layout: "post"
---

More and more frequently now I find myself trying to find SQL queries in a application that needs optimised or fixed. Rather than relying on mytop or a fancy application, you can watch the mysql process list, like so:

```bash
watch -n 0.5 --differences 'mysql -u<USERNAME> -p<PASSWORD> -h<HOST> -e"SHOW FULL PROCESSLIST" >> /tmp/processlist.txt'
```

Essentailly this snapshots the MySQL process list every 0.5 seconds and pipes it to the end of a file. After you trigger your slow query you can go back and view the file to see what the slow query was.
