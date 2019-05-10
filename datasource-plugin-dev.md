## 一种Grafana数据源插件开发的方案（纯后端）

### 背景
目前, Grafana官方已支持多种数据源, 例如: Prometheus, InfluxDB, MySQL, OpenTSDB, Elasticsearch等, 已经能满足绝大部分的需求. 但是, 在某些特定的情况下可能需要编写自己的数据源插件, 这要求开发者懂一些前端的东西, 笔者是个前端文盲, 因此另辟蹊径研究了纯后端的实现方案.

### 基本要求
- 后端开发
  
  Java, php, Golang, Python等都可以实现, 只要能满足需求即可, 本文以Golang为例.
- Nginx或者其他七层负载均衡器
  
  这个只要求会一些的简单请求参数判断就可以了, 本文以Nginx为例.

### 原理概述
Grafana前端(配置Panel来可视化数据, 这里选择配置好的InfluxDB数据源), 首先Panel(可以是Graph或者其他图表)会发送请求获取数据然后再进行渲染, 数据会通过配置好的InfluxDB数据源向实际的后端数据源获取, 此时我们在实际数据源和数据源插件之间增加一个Nginx来代理请求, 通过特定的查询语句的前缀来区分是否是我们自定义的请求, 如果是那么代理到我们自己的上游后端, 否则直接转到原来的后端(也就是真正的InfluxDB), 这样一来我们就完成了对Grafana前端的"欺骗", 届时我们自己的后端按照InfluxDB数据源要求的格式返回数据即可, 换作是Prometheus数据源活着的其他数据源也是类似的, 这样，就极大的扩展了支持的数据源, 因为后端就算不是对应的数据源也可以提供数据了.

### 实施
这里只提供一个简单的例子.

- 后端实现:
```
package main

import (
	"net/http"
)

const testData = `{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "test",
                    "tags": {
                        "cluster": "song",
                        "model": "cache",
                        "project": "test",
                        "protocol": "cache/Hint/web"
                    },
                    "columns": [
                        "time",
                        "total"
                    ],
                    "values": [
                        [
                            1557449700000,
                            null
                        ],
                        [
                            1557450000000,
                            null
                        ],
                        [
                            1557450300000,
                            null
                        ],
                        [
                            1557450600000,
                            null
                        ],
                        [
                            1557450900000,
                            null
                        ],
                        [
                            1557451200000,
                            null
                        ],
                        [
                            1557451500000,
                            null
                        ],
                        [
                            1557451800000,
                            null
                        ],
                        [
                            1557452100000,
                            null
                        ],
                        [
                            1557452400000,
                            null
                        ],
                        [
                            1557452700000,
                            null
                        ],
                        [
                            1557453000000,
                            null
                        ],
                        [
                            1557453300000,
                            null
                        ]
                    ]
                }
            ]
        }
    ]
}`

func testHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte(testData))
}

func main() {
	mux := http.NewServeMux()

	mux.HandleFunc("/test", testHandler)
	http.ListenAndServe(":12345", mux)
}
```

- Nginx配置
```
upstream influxdb
{
        server 127.0.0.1:8086;
}

upstream myifx
{
        server 192.168.10.1:12345;
}

server {
        listen 8086;
        server_name influxdb.ximply.com;

        set $myifx 0;
        if ($arg_q ~* (myprefix.*)) {
            set $myifx 1;
        }

        location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header REMOTE-HOST $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                
                if ($myifx = 1) {
                        proxy_pass http://myifx/test?q=$arg_q;
                        break;
                }
                proxy_pass http://influxdb;
        }

        #access_log off;
        access_log /home/www/log/ifx.access.log;
        error_log /home/www/log/error.log;
}
```

- Grafana Graph 面板
查询语句为: ```myprefix{"from":$__from,"to":$__to,","protocol":"test"} ```

注: 首先我们要选择InfluxDB作为数据源, 然后这里的 ```myprefix``` 一定要跟Nginx那边的参数判断保持一致, 这样才能进行区分, 另外查询语句可以自己定义格式, 我这里是比较通用的Json格式, 里面的参数可以加入Grafana本身的变量以及自己定义的变量, ```protocol``` 这个是用来区别具体请求的(例如获取CPU使用率, 获取内存使用率等, 根据自己的需求来定义). 

### 最后
该方案算是有点投机取巧, 不过在某些需要的情况下还是可以使用的, 可以作为开发自定义数据源插件的一个扩展吧, 如果有前端资源的话, 还是建议按照官方流程来做.
