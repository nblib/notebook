# 摘要
`syslog`是一个消息日志的标准.允许软件生成消息交由系统储存,再由别的软件进行传达和分析.

# 组成
* Facility : 设备标识,指明生成日志的软件标识
* Severity level : 日志级别
* Message : 消息
### Facility
表明生成日志的程序类型,不同类型可能会有不同的处理方式,下面是可用设备标识列表(定义在[RFC 3164](https://tools.ietf.org/html/rfc3164))
Facility code|	Keyword|	Description
---|---|---|
0|	kern|	kernel messages(内核消息)
1|	user|	user-level messages(用户级别消息)
2|	mail|	mail system(邮件系统)
3|	daemon|	system daemons(系统守护)
4|	auth|	security/authorization messages(安全消息)
5|	syslog|	messages generated internally by syslogd
6|	lpr|	line printer subsystem
7|	news|	network news subsystem
8|	uucp|	UUCP subsystem
9|		|clock daemon
10|	authpriv|	security/authorization messages
11|	ftp|	FTP daemon
12|	-|	NTP subsystem
13|	-|	log audit
14|	-|	log alert
15|	cron|	scheduling daemon
16|	local0|	local use 0 (local0)
17|	local1|	local use 1 (local1)
18|	local2|	local use 2 (local2)
19|	local3|	local use 3 (local3)
20|	local4|	local use 4 (local4)
21|	local5|	local use 5 (local5)
22|	local6|	local use 6 (local6)
23|	local7|	local use 7 (local7)

> facility code 和 关键字 在不同操作系统和不同syslog的实现是不统一的.

### Severity level
日志的级别(定义在[RFC 5424](https://tools.ietf.org/html/rfc5424))
Value|	Severity|	Keyword|	Deprecated keywords|	Description
---|---|---|---|---
0|	Emergency|	emerg|	panic[8]|	System is unusable.A panic condition.[9]
1|	Alert|	alert|		| Action must be taken immediately.A condition that should be corrected immediately, such as a corrupted system database.[9]
2|	Critical|	crit|		|Critical conditions, such as hard device errors.[9]
3|	Error|	err|	error[8]|	Error conditions.
4|	Warning|	warning|	warn[8]|	Warning conditions.
5|	Notice|	notice|		|Normal but significant conditions.Conditions that are not error conditions, but that may require special handling.[9]
6|	Informational|	info|		|Informational messages.
7|	Debug|	debug|		|Debug-level messages.Messages that contain information normally of use only when debugging a program.[9]

### Message
必须包含TAG和CONTENT,分别表示程序名称和详细消息.


