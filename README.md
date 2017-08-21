# stress

### 新增功能
`echo "POST http://www.baidu.com" | stress -v=1 -log_dir=log_dir attack -duration=5s -rate=1`

可以在 stress 后紧跟 `-v=1 -log_dir=log_dir` 实现打印`response body` 到目录 `log_dir`

A test tool to send random http GET/POST requests to server.
Fork from [Vegeta](https://github.com/tsenart/vegeta).

Stress is a versatile HTTP load testing tool built out of need to drill
HTTP services with a constant request rate or concurrency level.
It can be used both as a command line utility and a library.

Author: [@招牌疯子](http://weibo.com/819880808)  
Contact me: zp@buaa.us  

[![Build Status](https://travis-ci.org/buaazp/stress.svg?branch=master)](https://travis-ci.org/buaazp/stress) [![wercker status](https://app.wercker.com/status/5e44af92a3de33c3ccc4dfd385405887/s "wercker status")](https://app.wercker.com/project/bykey/5e44af92a3de33c3ccc4dfd385405887) [![GoDoc](https://godoc.org/github.com/buaazp/stress?status.png)](https://godoc.org/github.com/buaazp/stress)  

## Install
### Pre-compiled executables
Get them [here](https://github.com/buaazp/stress/releases).

### Source
You need go installed and `GOBIN` in your `PATH`. Once that is done, run the
command:

````
$ go get github.com/buaazp/stress
$ go install github.com/buaazp/stress
````

## Usage manual

````
➜ stress git:(master) ✗ stress -h
Usage: stress [globals] <command> [options]

global flags:
  -cpus=8 Number of CPUs to use

examples:
  echo "GET HOST:ww2.sinaimg.cn resize-type:crop.100.100.200.200.100 http://127.0.0.1:8088/bmiddle/50caec1agw1ef9myz5zhoj21ck0yggv6.jpg" | stress attack -duration=5s -rate=100 | tee results.bin | stress report
  echo "POST http://127.0.0.1:12345/ form:filename:5f189.jpeg" | stress attack -duration=5s -rate=1 | tee results.bin | stress report
  stress attack -targets=targets.txt > results.bin
  stress report -input=results.bin -reporter=json > metrics.json
  cat results.bin | stress report -reporter=plot > plot.html
````

#### -cpus
Specifies the number of CPUs to be used internally.
It defaults to the amount of CPUs available in the system.

### attack

````
➜ stress git:(master) ✗ stress attack -h
Usage of stress attack:
  -body="": Requests body file
  -c=10: Concurrency level
  -duration=10s: Duration of the test
  -header=: Request header
  -laddr=0.0.0.0: Local IP address
  -n=1000: Requests number
  -ordering="random": Attack ordering [sequential, random]
  -output="result.json": Output file
  -rate=50: Requests per second
  -redirects=10: Number of redirects to follow
  -targets="stdin": Targets file
  -timeout=0: Requests timeout
````

#### -rate
Specifies the requests per second rate to issue against
the targets. The actual request rate can vary slightly due to things like
garbage collection, but overall it should stay very close to the specified.

#### -duration
Specifies the amount of time to issue request to the targets.
The internal concurrency structure's setup has this value as a variable.
The actual run time of the test can be longer than specified due to the
responses delay.

#### -c
Specifies the concurrency level of attack. Concurrency level `-c` is conflict with `-rate`. You can't use them both in one stress test.

#### -n
Specifies the requests' number in one stress test. Use `-c` and `-n` to control amount of stress. Equal to use `-rate` and `-duration`.

#### -targets
Specifies the attack targets in a line separated file, defaulting to stdin.
The format should be as follows.

````
GET [Header_key:Header_value ...] Url [md5:response_body_md5_to_match]
POST [Header_key:Header_value ...] Url [[form:[filekey:]]BodyFile]
GET http://user:password@goku:9090/path/to
HEAD http://goku:9090/path/to/success
POST http://127.0.0.1:4869/upload form:5f189.jpeg
POST http://127.0.0.1:12345/ form:filename:5f189.jpeg
GET HOST:ww2.sinaimg.cn resize-type:crop.100.100.200.200.100 http://127.0.0.1:8088/bmiddle/50caec1agw1ef9myz5zhoj21ck0yggv6.jpg
GET http://127.0.0.1:4869/a.jpeg md5:5f189d8ec57f5a5a0d3dcba47fa797e2
...
````

#### -header
Specifies a request header to be used in all targets defined.
You can specify as many as needed by repeating the flag.

#### -laddr
Specifies the local IP address to be used.

#### -body
Specifies the file whose content will be set as the body of every request.

#### -ordering
Specifies the ordering of target attack. The default is `random` and
it will randomly pick one of the targets per request.
The other option is `sequential` and it round-robins through the list of
targets for each request.

#### -output
Specifies the output file to which the binary results will be written
to. Made to be piped to the report command input. Defaults to stdout.

#### -redirects
Specifies the max number of redirects followed on each request. The
default is 10.

#### -timeout
Specifies the timeout for each request. The default is 0 which disables
timeouts.

### report
````
➜ stress git:(master) ✗ stress report -h
Usage of stress report:
  -input="stdin": Input files (comma separated)
  -output="stdout": Output file
  -reporter="text": Reporter [text, json, plot]
````

#### -input
Specifies the input files to generate the report of, defaulting to stdin.
These are the output of stress attack. You can specify more than one (comma
separated) and they will be merged and sorted before being used by the
reports.

#### -output
Specifies the output file to which the report will be written to.

#### -reporter
Specifies the kind of report to be generated. It defaults to text.

##### text
````
Requests      [total]                   1200
Duration      [total]                   1.998307684s
Latencies     [mean, 50, 95, 99, max]   223.340085ms, 240.12234ms, 326.913687ms, 416.537743ms, 7.788103259s
Bytes In      [total, mean]             3714690, 3095.57
Bytes Out     [total, mean]             0, 0.00
Success       [ratio]                   55.42%
Status Codes  [code:count]              0:535  200:665
Error Set:
Get http://localhost:6060: dial tcp 127.0.0.1:6060: connection refused
Get http://localhost:6060: read tcp 127.0.0.1:6060: connection reset by peer
Get http://localhost:6060: dial tcp 127.0.0.1:6060: connection reset by peer
Get http://localhost:6060: write tcp 127.0.0.1:6060: broken pipe
Get http://localhost:6060: net/http: transport closed before response was received
Get http://localhost:6060: http: can't write HTTP request on broken connection
````

##### json
````
{
  "latencies": {
    "mean": 9093653647,
    "50th": 2401223400,
    "95th": 12553709381,
    "99th": 12604629125,
    "max": 12604629125
  },
  "bytes_in": {
    "total": 782040,
    "mean": 651.7
  },
  "bytes_out": {
    "total": 0,
    "mean": 0
  },
  "duration": 1998307684,
  "requests": 1200,
  "success": 0.11666666666666667,
  "status_codes": {
    "0": 1060,
    "200": 140
  },
  "errors": [
    "Get http://localhost:6060: dial tcp 127.0.0.1:6060: operation timed out"
  ]
}
````
##### plot
Generates an HTML5 page with an interactive plot based on
[Dygraphs](http://dygraphs.com).
Click and drag to select a region to zoom into. Double click to zoom
out.
Input a different number on the bottom left corner input field
to change the moving average window size (in data points).

![Plot](http://ww1.sinaimg.cn/large/4c422e03tw1egjnqkopjwj20lv0hjdhs.jpg)


## Usage (Library)

````
package main

import (
  stress "github.com/buaazp/stress/lib"
  "time"
  "fmt"
)

func main() {
  targets, _ := stress.NewTargets([]string{"GET http://localhost:9100/"})
  rate := uint64(100) //per second
  duration := 4 * time.Second
  concurrency := uint64(20)
  number := uint64(1000)

  results := stress.AttackRate(targets, rate, duration)
  metrics := stress.NewMetrics(results)

  fmt.Printf("Mean latency: %s", metrics.Latencies.Mean)

  results = stress.AttackConcy(targets, concurrency, number)
  metrics = stress.NewMetrics(results)

  fmt.Printf("Mean latency: %s", metrics.Latencies.Mean)
}
````

#### Limitations
There will be an upper bound of the supported `rate` which varies on the
machine being used.
You could be CPU bound (unlikely), memory bound (more likely) or
have system resource limits being reached which ought to be tuned for
the process execution. The important limits for us are file descriptors
and processes. On a UNIX system you can get and set the current
soft-limit values for a user.

````
$ ulimit -n # file descriptors
2560
$ ulimit -u # processes / threads
709
````
Just pass a new number as the argument to change it.

## Licence

Stress is under BSD license which is in the license file.  
But stress is forked form [Vegeta](https://github.com/tsenart/vegeta). The author of vegeta is [tsenart](https://github.com/tsenart) and vegeta is under MIT license. Thanks for his work.  

```
The MIT License (MIT)

Copyright (c) 2013, 2014 Tomás Senart

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

```
