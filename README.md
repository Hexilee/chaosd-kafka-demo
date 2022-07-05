## 构建 chaosd

下载[远程分支](https://github.com/Hexilee/chaosd/tree/kafka-set-retention-bytes)源码，运行 `make build` 构建得到可执行文件 `bin/chaosd`。

```bash
(kafka-set-retention-bytes)> make build
GO15VENDOREXPERIMENT="1" CGO_ENABLED=1 GOOS="" GOARCH="" go build -ldflags '-s -w -X 'github.com/chaos-mesh/chaosd/pkg/version.buildDate=2022-07-05T17:48:30Z' -X 'github.com/chaos-mesh/chaosd/pkg/version.gitCommit=2f2cb2b1c0a4e3c5397f39e95d96eebcb874e71f' -X 'github.com/chaos-mesh/chaosd/pkg/version.gitVersion=v1.1.1-26+2f2cb2b1c0a4e3'' -tags "" -o bin/chaosd ./cmd/main.go
# github.com/mattn/go-sqlite3
sqlite3-binding.c: In function ‘sqlite3SelectNew’:
sqlite3-binding.c:125801:10: warning: function may return address of local variable [-Wreturn-local-addr]
125801 |   return pNew;
       |          ^~~~
sqlite3-binding.c:125761:10: note: declared here
125761 |   Select standin;
       |          ^~~~~~~
GO15VENDOREXPERIMENT="1" CGO_ENABLED=1 GOOS="" GOARCH="" go build -o bin/tools/PortOccupyTool tools/PortOccupyTool.go
GO15VENDOREXPERIMENT="1" CGO_ENABLED=1 GOOS="" GOARCH="" go build -o bin/tools/FileTool tools/file/*.go
(kafka-set-retention-bytes)> ls -lah bin
total 54M
drwxr-xr-x 1 hexi hexi   42 Jul  6 01:49 ./
drwxr-xr-x 1 hexi hexi  312 Jul  4 00:19 ../
-rwxr-xr-x 1 hexi hexi  54M Jul  6 01:48 chaosd*
-rw-r--r-- 1 hexi hexi 116K Jul  6 01:49 chaosd.dat
drwxr-xr-x 1 hexi hexi   76 Jun 23 18:49 tools/
```

## 功能概览

执行 `bin/chaosd attack kafka -h` 可得功能介绍。

```bash
(kafka-set-retention-bytes)> bin/chaosd attack kafka -h
Kafka attack related commands

Usage:
  chaosd attack kafka [command]

Available Commands:
  fill        Fill kafka cluster with messages
  flood       Flood kafka cluster with messages
  io          Make kafka cluster non-writable/non-readable

Flags:
  -h, --help           help for kafka
  -T, --topic string   the topic to attack

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID

Use "chaosd attack kafka [command] --help" for more information about a command.
```

## 功能展示

### `flood`

```bash
(kafka-set-retention-bytes)> bin/chaosd attack kafka flood -h
Flood kafka cluster with messages

Usage:
  chaosd attack kafka flood [options] [flags]

Flags:
  -h, --help              help for flood
  -H, --host string       the host of kafka server (default "localhost")
  -p, --password string   the password of kafka client
  -P, --port uint16       the port of kafka server (default 9092)
  -s, --size uint         the size of each message (default 1024)
  -t, --threads uint      the numbers of worker threads (default 100)
  -u, --username string   the username of kafka client

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
  -T, --topic string       the topic to attack
      --uid string         the experiment ID
```

如无特殊配置，直接运行命令给 topic `test` 加负载。

```bash
bin/chaosd attack kafka flood -T test
[2022/07/06 02:47:12.575 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 555, failed: 0"] [thread=thread-62]
[2022/07/06 02:47:12.575 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 553, failed: 0"] [thread=thread-27]
[2022/07/06 02:47:12.576 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 555, failed: 0"] [thread=thread-65]
[2022/07/06 02:47:12.576 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 558, failed: 0"] [thread=thread-38]
[2022/07/06 02:47:12.577 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 554, failed: 0"] [thread=thread-51]
[2022/07/06 02:47:12.577 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 555, failed: 0"] [thread=thread-59]
[2022/07/06 02:47:12.577 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 553, failed: 0"] [thread=thread-48]
[2022/07/06 02:47:12.578 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 556, failed: 0"] [thread=thread-80]
[2022/07/06 02:47:12.578 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 554, failed: 0"] [thread=thread-29]
[2022/07/06 02:47:12.578 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 554, failed: 0"] [thread=thread-30]
[2022/07/06 02:47:12.579 +08:00] [INFO] [asm_amd64.s:1571] ["succeeded: 553, failed: 0"] [thread=thread-50]
...
```

运行过程中可使用 `top` 或 `htop` 查看 kafka server 的资源占用情况。

### `fill`

```bash
(kafka-set-retention-bytes)> bin/chaosd attack kafka fill -h
Fill kafka cluster with messages

Usage:
  chaosd attack kafka fill [options] [flags]

Flags:
  -h, --help                help for fill
  -H, --host string         the host of kafka server (default "localhost")
  -m, --max-bytes uint      the max bytes to fill (default 17179869184)
  -p, --password string     the password of kafka client
  -P, --port uint16         the port of kafka server (default 9092)
  -r, --reload-cmd string   the command to reload kafka config
  -s, --size uint           the size of each message (default 4096)
  -u, --username string     the username of kafka client

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
  -T, --topic string       the topic to attack
      --uid string         the experiment ID
```

如无特殊配置，直接运行命令填满 topic `test`。

> 此命令可能需要 sudo 权限

```bash
(kafka-set-retention-bytes)> bin/chaosd attack kafka fill -T test -m 1000_000 -r "systemctl restart kafka"
[2022/07/06 02:50:55.852 +08:00] [INFO] [kafka.go:197] ["write 131 kB in 5.32314ms"]
[2022/07/06 02:50:55.864 +08:00] [INFO] [kafka.go:197] ["write 262 kB in 17.334654ms"]
[2022/07/06 02:50:55.865 +08:00] [INFO] [kafka.go:197] ["write 393 kB in 17.900474ms"]
[2022/07/06 02:50:55.866 +08:00] [INFO] [kafka.go:197] ["write 524 kB in 18.606828ms"]
[2022/07/06 02:50:55.866 +08:00] [INFO] [kafka.go:197] ["write 655 kB in 19.282199ms"]
[2022/07/06 02:50:55.867 +08:00] [INFO] [kafka.go:197] ["write 786 kB in 19.918358ms"]
[2022/07/06 02:50:55.868 +08:00] [INFO] [kafka.go:197] ["write 918 kB in 20.558703ms"]
[2022/07/06 02:50:55.868 +08:00] [INFO] [kafka.go:197] ["write 1.0 MB in 21.134191ms"]
Attack kafka successfully, uid: 564e9251-c43b-49f7-9539-c20a479a0b55
```

执行后 topic `test` 会填入 1000_000 bytes 的数据，并且更改 kafka 配置中的 `log.retention.bytes` 为 1000_000，然后使用 `systemctl restart kafka` 命令重启加载配置。

我们查看 kafka 配置文件和运行状态可看出 `log.retention.bytes` 已发生更改，并且 kafka server 6s 之前发生过重启。

```bash
(kafka-set-retention-bytes)> cat /etc/kafka/server.properties
broker.id = 0
num.network.threads = 3
num.io.threads = 8
socket.send.buffer.bytes = 102400
socket.receive.buffer.bytes = 102400
socket.request.max.bytes = 104857600
log.dirs = /var/lib/kafka
num.partitions = 1
num.recovery.threads.per.data.dir = 1
offsets.topic.replication.factor = 1
transaction.state.log.replication.factor = 1
transaction.state.log.min.isr = 1
log.retention.hours = 168
log.retention.bytes = 1000000
log.segment.bytes = 1073741824
log.retention.check.interval.ms = 300000
zookeeper.connect = localhost:2181
zookeeper.connection.timeout.ms = 18000
group.initial.rebalance.delay.ms = 0
(kafka-set-retention-bytes)> systemctl status kafka
· kafka.service - Kafka publish-subscribe messaging system
     Loaded: loaded (/usr/lib/systemd/system/kafka.service; enabled; vendor preset: disabled)
     Active: active (running) since Wed 2022-07-06 02:54:30 CST; 6s ago
   Main PID: 294145 (java)
      Tasks: 79 (limit: 38163)
     Memory: 336.1M
        CPU: 3.995s
     CGroup: /system.slice/kafka.service
             └─294145 /usr/bin/java -Xmx1G -Xms1G -server -XX:+UseCompressedOops -XX:+UseG1GC -XX:+DisableExplicitGC -D>
```

### `io`

```bash
(kafka-set-retention-bytes)> bin/chaosd attack kafka io -h
Make kafka cluster non-writable/non-readable

Usage:
  chaosd attack kafka io [options] [flags]

Flags:
  -c, --config string   the path of server config (default "/etc/kafka/server.properties")
  -h, --help            help for io
  -r, --non-readable    make kafka cluster non-readable
  -w, --non-writable    make kafka cluster non-writable

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
  -T, --topic string       the topic to attack
      --uid string         the experiment ID
```

如无特殊配置，直接运行命令 topic `test` 的 log 文件无法读写。

> 此命令可能需要 sudo 权限

```bash
(kafka-set-retention-bytes)> bin/chaosd attack kafka io -T test -rw
Attack kafka successfully, uid: 0503b168-4c42-493b-83e6-04313473f435
```

注入后可查看 topic `test` 的 log 目录，我们可以看出目录 `test-0` 和里面的 log 文件均无读写权限。

> 此命令可能需要 sudo 权限

```bash
(kafka-set-retention-bytes)> ls -lah /var/lib/kafka/test-0/
total 1022M
d--x--x--x 1 kafka kafka   300 Jul  6 02:54 .
drwxr-xr-x 1 kafka kafka   300 Jul  6 03:01 ..
-rw-r--r-- 1 kafka kafka   10M Jul  6 02:54 00000000000026993693.index
---------- 1 kafka kafka 1018M Jul  6 02:54 00000000000026993693.log
-rw-r--r-- 1 kafka kafka   10M Jul  6 02:54 00000000000026993693.timeindex
-rw-r--r-- 1 kafka kafka    10 Jul  6 02:54 00000000000027968805.snapshot
-rw-r--r-- 1 kafka kafka    15 Jul  6 02:49 leader-epoch-checkpoint
-rw-r--r-- 1 kafka kafka    43 May  6 11:16 partition.metadata
```

此时 kafka server 并不会立刻无法读写，可以试着给它加一点负载：

```bash
(kafka-set-retention-bytes)> sudo bin/chaosd attack kafka fill -T test -m 3000_000_000 -r "systemctl restart kafka"
...
[2022/07/06 03:03:10.040 +08:00] [INFO] [kafka.go:197] ["write 5.1 MB in 111.410495ms"]
[2022/07/06 03:03:10.041 +08:00] [INFO] [kafka.go:197] ["write 5.2 MB in 113.008611ms"]
[2022/07/06 03:03:10.043 +08:00] [INFO] [kafka.go:197] ["write 5.4 MB in 114.570968ms"]
[2022/07/06 03:03:10.045 +08:00] [INFO] [kafka.go:197] ["write 5.5 MB in 116.415197ms"]
[2022/07/06 03:03:10.047 +08:00] [INFO] [kafka.go:197] ["write 5.6 MB in 118.19826ms"]
[2022/07/06 03:03:10.048 +08:00] [INFO] [kafka.go:197] ["write 5.8 MB in 119.790419ms"]
[2022/07/06 03:03:10.050 +08:00] [INFO] [kafka.go:197] ["write 5.9 MB in 121.237723ms"]
[2022/07/06 03:03:10.051 +08:00] [INFO] [kafka.go:197] ["write 6.0 MB in 122.734107ms"]
[2022/07/06 03:03:10.053 +08:00] [INFO] [kafka.go:197] ["write 6.2 MB in 124.806888ms"]
[2022/07/06 03:03:10.055 +08:00] [INFO] [kafka.go:197] ["write 6.3 MB in 126.294494ms"]
[2022/07/06 03:03:10.057 +08:00] [INFO] [kafka.go:197] ["write 6.4 MB in 128.330724ms"]
[2022/07/06 03:03:10.059 +08:00] [INFO] [kafka.go:197] ["write 6.6 MB in 130.052328ms"]
[2022/07/06 03:03:10.060 +08:00] [INFO] [kafka.go:197] ["write 6.7 MB in 131.909499ms"]
[2022/07/06 03:03:10.062 +08:00] [INFO] [kafka.go:197] ["write 6.8 MB in 133.41762ms"]
[2022/07/06 03:03:10.063 +08:00] [INFO] [kafka.go:197] ["write 6.9 MB in 134.875465ms"]
Error: write messages: [56] Kafka Storage Error: disk error when trying to access log file on the disk
```