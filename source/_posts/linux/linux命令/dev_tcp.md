---
title: Linux Bash /dev/tcp/HOST/PORT
date: 2018-12-25 00:00:00
tags: 
- linux
categories: 
- linux
---
> bash中的特殊命令
<!-- more -->
```
exec 8<>/dev/tcp/192.168.106.213/9200
echo -e "GET _cat/indices HTTP/1.1\r\nhost: http://127.0.0.1\r\nConnection: close\r\n\r\n" >&8
cat <&8
exec 8<&-
```