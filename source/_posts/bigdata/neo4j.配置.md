
---
title: neo4j 3.5.4 配置
date: 2019-07-08 00:00:00
tags:
- neo4j
- ogg
categories:
- tool
---

> neo4j 配置
<!-- more -->

## neo4j 配置
```
dbms.directories.import=import
dbms.allow_upgrade=true
dbms.connectors.default_listen_address=192.168.11.117
dbms.connector.bolt.enabled=true
dbms.connector.bolt.listen_address=:7687
dbms.connector.http.enabled=true
dbms.connector.http.listen_address=:7474
dbms.connector.https.enabled=true
dbms.mode=HA
ha.server_id=3
ha.initial_hosts=192.168.11.119:5001,192.168.11.118:5001,192.168.11.117:5001
dbms.security.allow_csv_import_from_file_urls=true
dbms.jvm.additional=-XX:+UseG1GC
dbms.jvm.additional=-XX:-OmitStackTraceInFastThrow
dbms.jvm.additional=-XX:+AlwaysPreTouch
dbms.jvm.additional=-XX:+UnlockExperimentalVMOptions
dbms.jvm.additional=-XX:+TrustFinalNonStaticFields
dbms.jvm.additional=-XX:+DisableExplicitGC
dbms.jvm.additional=-Djdk.tls.ephemeralDHKeySize=2048
dbms.jvm.additional=-Djdk.tls.rejectClientInitiatedRenegotiation=true
dbms.windows_service_name=neo4j
dbms.jvm.additional=-Dunsupported.dbms.udc.source=tarball
```