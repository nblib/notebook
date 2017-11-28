# 概述
log4go官方没有提供`syslog`的支持,但是提供了`socket`支持,所以只需要通过`socket`简单修改即可.

官方的`socket`输出通过转化为json,所以,新建一个go文件`syslog.go`在log4go的目录下,和socket在同一个目录中:
``` 
package log4go

import (
	"net"
	"fmt"
	"os"
	"bytes"
	"strconv"
)

/*
severity_code
 */
const (
	SYSLOG_CRITICAL = 2
	SYSLOG_ERROR    = 3
	SYSLOG_WARN     = 4
	SYSLOG_INFO     = 6
	SYSLOG_DEBUG    = 7

	SYSLOG_NOTICE = 5
)

/*
	facility_code
 */
const FACILITY = 1

func NewSysLogWriter(proto, hostport string) SocketLogWriter {
	sock, err := net.Dial(proto, hostport)
	if err != nil {
		fmt.Fprintf(os.Stderr, "NewSocketLogWriter(%q): %s\n", hostport, err)
		return nil
	}

	w := SocketLogWriter(make(chan *LogRecord, LogBufferLength))

	go func() {
		defer func() {
			if sock != nil && proto == "tcp" {
				sock.Close()
			}
		}()

		for rec := range w {
			results := buildSyslog(rec)
			_, err = sock.Write(results)
			if err != nil {
				fmt.Fprint(os.Stderr, "SocketLogWriter(%q): %s", hostport, err)
				return
			}
		}
	}()

	return w
}
func buildSyslog(rec *LogRecord) []byte {
	var results *bytes.Buffer
	//toString
	results = new(bytes.Buffer)
	var severity, facility, priority int
	var createdStr, logSource, program, message string

	severity = determineSeverity(rec.Level)
	facility = FACILITY
	priority = calculatePriority(facility, severity)

	createdStr = rec.Created.Format("Jan 2 15:04:05")
	//var js []byte
	logSource, err := os.Hostname()
	if err != nil {
		logSource = "UNKNOWN-HOST"
	}
	program = rec.Source
	message = rec.Message

	results.WriteString("<")
	results.WriteString(strconv.Itoa(priority))
	results.WriteString(">")
	results.WriteString(createdStr)
	results.WriteString(" ")
	results.WriteString(logSource)
	results.WriteString(" ")
	results.WriteString(program)
	results.WriteString(": ")
	results.WriteString(message)

	return results.Bytes()
}
func calculatePriority(facility, severity int) int {
	return facility<<3 | severity
}
func determineSeverity(lev level) int {
	switch lev {
	case DEBUG:
		return SYSLOG_DEBUG
	case INFO:
		return SYSLOG_INFO
	case WARNING:
		return SYSLOG_WARN
	case ERROR:
		return SYSLOG_ERROR
	case CRITICAL:
		return SYSLOG_CRITICAL
	}
	return SYSLOG_NOTICE
}
```
输出的格式为:
``` 
<pri>Month day hh:mm:ss <hostname> <program>: <message>
```

# 使用例子
``` 
func main() {
	log := l4g.NewLogger()
	log.AddFilter("network", l4g.FINEST, l4g.NewSysLogWriter("udp", "192.168.233.129:4699"))

	// Run `nc -u -l -p 12124` or similar before you run this to see the following message
	log.Info("The time is now: %s", time.Now().Format("15:04:05 MST 2006/01/02"))
	log.Error("this is error log")
	log.Critical("this is Critical log")
	log.Warn("this is warn log")
	log.Debug("this is debug log")
	// This makes sure the output stream buffer is written
	log.Close()
	time.Sleep(1000*1000*10)
}
```