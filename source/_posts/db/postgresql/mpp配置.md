---
title: mpp配置
date: 2019-10-11 01:00:00
tags: 
- postgresql 
categories: 
- db
---


>mpp配置

<!-- more -->

## bireme

```
[root@localhost etc]# pwd
/opt/bireme-2.0.0-alpha-1/etc
[root@localhost etc]# ls
config.properties  config.properties.bak  debezium1.properties  kafka_client_jaas.conf  log4j2.xml  maxwell1.properties  mysql.propertie
[root@localhost etc]# cat config.properties
# target database where the data will sync into.
target.url = jdbc:postgresql://10.24.70.135:5432/hchdata
target.user = gpadmin
target.passwd = gpadmin

# data source name list, separated by comma.
data_source = mysql

# data source "mysql1" type
mysql.type = maxwell
# kafka server which maxwell write binlog into.
mysql.kafka.server = 10.24.70.127:9092,10.24.70.128:9092,10.24.70.134:9092
# kafka topic which maxwell write binlog into.
mysql.kafka.topic = maxwell
# kafka groupid used for consumer.
mysql.kafka.groupid = bireme
mysql.kafka.security.protocol=SASL_PLAINTEXT
mysql.kafka.sasl.mechanism=PLAIN
# data source "debezium1"
#debezium1.type = debezium
# kafka server which debezium write into.
#debezium1.kafka.server = 127.0.0.1:9092 
# kafka groupid used for consumer.
#debezium1.kafka.groupid = bireme

# number of threads used for pipeline to drive the porcess
pipeline.thread_pool.size = 5

# number of threads used to transform data source record into target format.
transform.thread_pool.size = 10

# number of threads used to generate load tasks.
merge.thread_pool.size = 10
# interval of generating a load task in milliseconds.
merge.interval = 10000
# max tuple size for a load task
merge.batch.size = 50000

# JDBC connection pool size of target database.
loader.conn_pool.size = 10
# queue size of task for each table which is waiting for load.
loader.task_queue.size = 2

# application performance monitor report type: "none", "console", "jmx"
metrics.reporter=jmx
# interval of console APM reporter.
metrics.reporter.console.interval = 10

# set the IP address for bireme state server.
state.server.addr = 0.0.0.0
# set the port for bireme state server.
state.server.port = 8080

[root@localhost etc]# cat mysql.properties 
infoveriplatform.tb_car_record=public.tb_car_record
infoveriplatform.tb_control_car=public.tb_control_car
infoveriplatform.tb_control_person=public.tb_control_person
infoveriplatform.tb_key_car_record=public.tb_key_car_record
infoveriplatform.tb_key_person_record=public.tb_key_person_record
infoveriplatform.tb_person_record=public.tb_person_record
infoveriplatform.tb_security_clearance=public.tb_security_clearance
infoveriplatform.sys_dept=public.sys_dept
infoveriplatform.tb_client_device=public.tb_client_device
[root@localhost etc]# vi mysql.properties 
[root@localhost etc]# cat mysql.properties 
infoveriplatform.tb_car_record=public.tb_car_record
infoveriplatform.tb_control_car=public.tb_control_car
infoveriplatform.tb_control_person=public.tb_control_person
infoveriplatform.tb_key_car_record=public.tb_key_car_record
infoveriplatform.tb_key_person_record=public.tb_key_person_record
infoveriplatform.tb_person_record=public.tb_person_record
infoveriplatform.tb_security_clearance=public.tb_security_clearance
infoveriplatform.sys_dept=public.sys_dept
infoveriplatform.tb_client_device=public.tb_client_device
infoveriplatform.tb_face_record=public.tb_face_record
[root@localhost etc]# cat kafka_client_jaas.conf 
KafkaClient{
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin@123";
};

[root@localhost bireme-2.0.0-alpha-1]# cd bin
[root@localhost bin]# ls
bireme
[root@localhost bin]# cat bireme 
#!/usr/bin/env bash
# Copyright HashData. All Rights Reserved.
NAME="bireme"
DESC="bireme service"
BINDIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
TOPDIR="$(cd "$(dirname "${BINDIR}")" && pwd)"

EXEC="$(which jsvc)"

if [ -z "${EXEC}" ]; then
  echo 'command "jsvc" is not found.' >&2
  exit 1
fi

if [ -z "${JAVA_HOME}" ]; then
  echo '"JAVA_HOME" is not set.' >&2
  exit 1
fi

CLASS_PATH="${TOPDIR}/lib/*"

CLASS="cn.hashdata.bireme.Bireme"

CMD=$1

shift

ARGS="$*"
PID="/tmp/$NAME.pid"

# System.out writes to this file...
LOG_OUT="${TOPDIR}/logs/$NAME.out"

# System.err writes to this file...
LOG_ERR="${TOPDIR}/logs/$NAME.err"

HEAP_DUMP="${TOPDIR}/logs/$NAME.heapdump"

LOG_GC="${TOPDIR}/logs/$NAME.gc"

# See the following page for extensive details on setting
# up the JVM to accept JMX remote management:
# http://java.sun.com/javase/6/docs/technotes/guides/management/agent.html
# by default we allow local JMX connections

println() {
  if [ "$CMD" = "start" ]
  then
    echo "$1" >&2
  fi
}

if [ -z "$JXM_LOCALONLY" ]
then
    JXM_LOCALONLY=false
fi

if [ -z "$JMX_DISABLE" ] || [ "$JMX_DISABLE" = 'false' ]
then
  println "Bireme JMX enabled by default" >&2
  if [ -z "$JMX_PORT" ]
  then
    JMX="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=$JXM_LOCALONLY"
  else
    if [ -z "$JMX_AUTH" ]
    then
      JMX_AUTH=false
    fi
    if [ -z "$JMX_SSL" ]
    then
      JMX_SSL=false
    fi
    if [ -z "$JMX_LOG4J" ]
    then
      JMX_LOG4J=true
    fi
    println "Bireme remote JMX Port set to $JMX_PORT" >&2
    println "Bireme remote JMX authenticate set to $JMX_AUTH" >&2
    println "Bireme remote JMX ssl set to $JMX_SSL" >&2
    println "Bireme remote JMX log4j set to $JMX_LOG4J" >&2
    JMX="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=$JMX_PORT -Dcom.sun.management.jmxremote.authenticate=$JMX_AUTH -Dcom.sun.management.jmxremote.ssl=$JMX_SSL -Dzookeeper.jmx.log4j.disable=$JMX_LOG4J"
  fi
else
    println "JMX disabled by user request" >&2
fi

if [ "$MAX_HEAP" ]
then
  println "Bireme maximum heap size is ${MAX_HEAP}m" >&2
  HEAP="-Xmx${MAX_HEAP}m"
fi


jsvc_exec() {
  cd "${TOPDIR}" || exit 1
    "${EXEC}" -cwd "${TOPDIR}" -home "${JAVA_HOME}" -cp "${CLASS_PATH}" \
    ${JMX} ${HEAP} \
    -Dlog4j.configurationFile=${TOPDIR}/etc/log4j2.xml \
    -Djava.security.auth.login.config=/opt/bireme-2.0.0-alpha-1/etc/kafka_client_jaas.conf \
    -XX:+UseG1GC -XX:+UseStringDeduplication \
    -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${HEAP_DUMP} \
    -XX:-PrintGCDetails -XX:-PrintGCTimeStamps -XX:-UseGCLogFileRotation \
    -XX:NumberOfGCLogFiles=2 -XX:GCLogFileSize=1024K -Xloggc:${LOG_GC} \
    -jvm server -wait 60 -outfile "${LOG_OUT}" -errfile "${LOG_ERR}" \
    -pidfile "${PID}" $1 "${CLASS}" ${ARGS}
}

case "${CMD}" in
  start)
    echo "Starting the $DESC..."

    # Start the service
    if ! jsvc_exec; then
      echo "Failed to start $DESC"
      exit 1
    fi

    echo "The $DESC has started."
  ;;
  stop)
    if [ -f "$PID" ]; then
      echo "Stopping the $DESC..."

      # Stop the service
      if ! jsvc_exec "-stop"; then
        echo "Failed to stop $DESC"
        exit 1
      fi

      echo "The $DESC has stopped."
    else
      echo "Daemon not running, no action taken"
      exit 1
    fi
  ;;
  restart)
    if [ -f "$PID" ]; then
      echo "Restarting the $DESC..."

      # Stop the service
      jsvc_exec "-stop"

      # Start the service
      if ! jsvc_exec; then
	    echo "Failed to start $DESC"
	    exit 1
	  fi

	  echo "The $DESC has restarted."
    else
      echo "Daemon not running, no action taken"
      exit 1
    fi
      ;;
  *)
    echo "Usage: $0 {start|stop|restart}" >&2
    exit 1
  ;;
esac
[root@localhost bin]# pwd
/opt/bireme-2.0.0-alpha-1/bin





































```

## maxwell

```






[root@localhost maxwell-1.22.1]# cat config
user=maxwell
password=maxwell
host=10.24.70.85
replication_host=10.24.70.25
replication_user=maxwell
replication_password=Maxwell@123
include_dbs=inforveriplatform
include_tables=tb_car_record,tb_control_car,tb_control_person,tb_person_record,tb_key_car_record,tb_key_person_record,tb_security_clearance,tb_client_device,sys_dept,tb_face_record
producer=kafka
kafka_topic=maxwell
kafka.bootstrap.servers=10.24.70.127:9092,10.24.70.128:9092,10.24.70.134:9092
kafka.security.protocol=SASL_PLAINTEXT
kafka.sasl.mechanism=PLAIN
kafka_partition_hash=murmur3
producer_partition_by=primary_key
[root@localhost maxwell-1.22.1]# pwd
/opt/maxwell-1.22.1

[root@localhost maxwell-1.22.1]# ls
bin  config  config.md  config.properties.example  kafka_client_jaas.conf  kinesis-producer-library.properties.example  lib  LICENSE  log4j2.xml  logs  quickstart.md  README.md
[root@localhost maxwell-1.22.1]# cat kafka_client_jaas.conf 
KafkaClient{
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin@123";
};


[root@localhost maxwell-1.22.1]# cd bin
[root@localhost bin]# ls
maxwell  maxwell-benchmark  maxwell-bootstrap  maxwell-docker
[root@localhost bin]# cat maxwell
#!/bin/bash

set -e

base_dir="$(dirname "$0")/.."
lib_dir="$base_dir/lib"
lib_dir_development="$base_dir/target/lib"
if [ ! -e "$lib_dir" -a -e "$lib_dir_development" ]; then
	lib_dir="$lib_dir_development"
	CLASSPATH="$CLASSPATH:$base_dir/target/classes"
fi

CLASSPATH="$CLASSPATH:$lib_dir/*"

KAFKA_VERSION="1.0.0"

function use_kafka() {
	wanted="$1"
	# disambiguate versions into the latest,
	# e.g. asking for 0.10 means you want the
	# latest 0.10.x release
	case "$wanted" in
		0.11)
		KAFKA_VERSION=0.11.0.1
		;;

		0.10)
		KAFKA_VERSION=0.10.2.1
		;;

		*)
		KAFKA_VERSION="$wanted"
		;;
	esac
}

for key in "$@"
do
	case "$key" in
		--kafka_version)
		use_kafka "$2"
		;;

		--kafka_version=*)
		use_kafka "${key#*=}"
		;;

        --daemon)
		DAEMON_MODE="true"
		DAEMON_NAME="MaxwellDaemon"
		;;

	esac
done

if [ "x$DAEMON_MODE" = "xtrue" ]; then
    # Log directory to use
    if [ "x$LOG_DIR" = "x" ]; then
        LOG_DIR="$base_dir/logs"
    fi
    # Create logs directory
    if [ ! -d "$LOG_DIR" ]; then
        mkdir -p "$LOG_DIR"
    fi
    # Console output file when maxwell runs as a daemon
    CONSOLE_OUTPUT_FILE=$LOG_DIR/$DAEMON_NAME.out
    echo "Redirecting STDOUT to $CONSOLE_OUTPUT_FILE"
fi

kafka_client_dir="$lib_dir/kafka-clients"
kafka_client_jar="$(ls -1 "$kafka_client_dir/kafka-clients-$KAFKA_VERSION"*.jar 2>/dev/null || true)"
if [ -z "$kafka_client_jar" -o "$(echo "$kafka_client_jar" | wc -l)" -gt 1 ]; then

	if [ -z "$kafka_client_jar" ]; then
		echo "Error: No matches for kafka version: $KAFKA_VERSION"
	else
		echo "Error: Multiple matches for kafka version: $KAFKA_VERSION"
	fi
	echo "Supported versions:"
	ls -1 "$kafka_client_dir" | sed -e 's/^kafka-clients-/ - /' -e 's/\.jar$//'
	exit 1
else
	echo "Using kafka version: $KAFKA_VERSION"
	CLASSPATH="$CLASSPATH:$kafka_client_jar"
fi

if [ -z "$JAVA_HOME" ]; then
	JAVA="java"
else
	JAVA="$JAVA_HOME/bin/java"
fi

SASL_OPTS="-Djava.security.auth.login.config=/opt/maxwell-1.22.1/kafka_client_jaas.conf"

# Launch mode
if [ "x$DAEMON_MODE" = "xtrue" ]; then
    nohup $JAVA $JAVA_OPTS $SASL_OPTS -Dfile.encoding=UTF-8 -Dlog4j.shutdownCallbackRegistry=com.djdch.log4j.StaticShutdownCallbackRegistry -cp $CLASSPATH com.zendesk.maxwell.Maxwell "$@" > "$CONSOLE_OUTPUT_FILE" 2>&1 < /dev/null &
else
    exec $JAVA $JAVA_OPTS $SASL_OPTS -Dfile.encoding=UTF-8 -Dlog4j.shutdownCallbackRegistry=com.djdch.log4j.StaticShutdownCallbackRegistry -cp $CLASSPATH com.zendesk.maxwell.Maxwell "$@"
fi
[root@localhost bin]# 









```

## mysql2pgsql(RDS_DBSYNC)

```




[root@localhost soft]# pwd
/opt/soft
[root@localhost soft]# ls
bireme-2.0.0-alpha-1.tar.gz  __MACOSX  maxwell-1.22.1.tar.gz  mysql2pgsql.bin.el7.20171213.zip
[root@localhost soft]# 


[root@localhost bin]# cat table_list.txt 
tb_face_record: select * from tb_face_record
[root@localhost bin]# cat my.cfg
[src.mysql]
host = "10.24.70.25"
port = "3306"
user = "root"
password = "123456"
db = "infoveriplatform"
encodingdir = "share"
encoding = "utf8"


[desc.pgsql]
connect_string = "host=10.24.70.135 dbname=hchdata port=5432 user=gpadmin password=gpadmin"
target_schema = "public"
ignore_copy_error_count_each_table = "0"

[root@localhost bin]# ./mysql2pgsql -l table_list.txt -j 10
ignore copy error count 0 each table
-- Adding table: tb_face_record
Starting data sync
Query to get source data for target table tb_face_record: select * from tb_face_record 
-- Reference DDL to create the target table:
CREATE TABLE tb_face_record (id int8, img_id text, sbid text, checkevent_id text, sfzh text, tpdz text, tpscdz text, picture_upload int4, is_upload int4, create_time timestamp, temp1 text, temp2 text, temp3 text, temp4 text, checkpoint_id text, inspector_id text, inspector_name text, company_name text, name text, key_person int4, feature_id text, score float4, is_deal int4, tpmc text) with (APPENDONLY=true, ORIENTATION=column, COMPRESSTYPE=zlib, COMPRESSLEVEL=1, BLOCKSIZE=1048576, OIDS=false) DISTRIBUTED BY (<distribution key>) PARTITION BY RANGE (<partition key>) (START (date '<YYYY-MM-DD>') INCLUSIVE END (date '<YYYY-MM-DD>') EXCLUSIVE EVERY (INTERVAL '<1 month>' ));

thread 6 migrate task 0 table infoveriplatform.tb_face_record 5386127 rows complete, time cost 30393.893 ms
Number of rows migrated: 5386127 (number of source tables' rows: 5386127) 
Data sync time cost 30481.859 ms



```