---
layout: post
title: prometheus初探
author: Fish-pro
tags:
- prometheus
date: 2021-01-01 13:56 +0800
---
大家对架构应该已经很了解了，prometheus server通过采集程序，获取应用的监控指标，存储到Time Series DB，通过web ui或者grafana对接server，根据prom特定的表达式，查询展示对应值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/366291b8e974439b967e63e609015958.png)


## Targets

```yaml
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'exporter'
    static_configs:
    - targets: ['127.0.0.1:9100']
  - job_name: 'pushgateway'
    static_configs:
    - targets: ['127.0.0.1:9091']
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a07e4b1d26243d9baff0b1e63664e43.png)


## node_exporter

![在这里插入图片描述](https://img-blog.csdnimg.cn/87e19d2e2fe34e9e84da0a4449187f92.png)


## pushgateway

为了防止 pushgateway 重启或意外挂掉，导致数据丢失，我们可以通过 `-persistence.file` 和 `-persistence.interval` 参数将数据持久化下来。

prometheus 每次从 PushGateway 拉取的数据，并不是拉取周期内用户推送上来的所有数据，而是最后一次 push 到 PushGateway 上的数据，
所以**推荐设置推送时间小于或等于 prometheus 拉取的时间**，这样保证每次拉取的数据是最新 Push 上来的。

```shell
#!/bin/bash

instance_name=`hostname -f | cut -d'.' -f1` # 本机名称

if [$instance_name == "localhost"];then #要求机器名不能为localhost
echo "Mush FQDN hostname"
exit 1
fi

# for waitting connections
count_netstat_wait_connections=`netstat -an | grep -i wait | wc -l`

echo "$count_netstat_wait_connections"

echo "count_netstat_wait_connections $count_netstat_wait_connections" | curl --data-binary @- http://localhost:9091/metrics/job/pushgateway/instance/$instance_name
```

## metrics type

+ Counter (只增不减的计数器)
+ Gauge (可增可减的仪表盘)
+ Summary (分析数据分布情况)
+ Histogram (分析数据分布情况)

### counter

Counter 一个累加指标数据，这个值随着时间只会逐渐的增加，比如程序完成的总任务数量，运行错误发生的总次数。常见的还有交换机中snmp采集的数据流量也属于该类型，代表了持续增加的数据包或者传输字节累加值。

### gauge

Gauge代表了采集的一个单一数据，这个数据可以增加也可以减少，比如CPU使用情况，内存使用量，硬盘当前的空间容量等等

### histogram and summary

场景：在大多数情况下人们都倾向于使用某些量化指标的平均值，例如CPU的平均使用率、页面的平均响应时间。这种方式的问题很明显，以系统API调用的平均响应时间为例：如果大多数API请求都维持在100ms的响应时间范围内，而个别请求的响应时间需要5s，那么就会导致某些WEB页面的响应时间落到中位数的情况，而这种现象被称为长尾问题。为了区分是平均的慢还是长尾的慢，最简单的方式就是按照请求延迟的范围进行分组。例如，统计延迟在010ms之间的请求数有多少而1020ms之间的请求数又有多少。通过这种方式可以快速分析系统慢的原因。Histogram和Summary都是为了能够解决这样问题的存在

通过Histogram和Summary类型的监控指标，我们可以快速了解监控样本的分布情况

## query function

### rate()--->counter

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-scRmjQST-1660630095768)(./rate.png)]

```mathematica
rate(node_network_receive_bytes_total[1m])
```

网络接收量一分钟以内的增量总量，除以60秒的数量

### increase()--->counter

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-CnqwM1tb-1660630095769)(./increase.png)]

使用同rate函数

```mathematica
rate(node_network_receive_bytes_total[1m])
```

网络接收量一分钟以内的增量总量

### sum() by (label)

```mathematica
sum(rate(node_network_receive_bytes_total[1m]))
```

将多条线相加，可以根据标签拆分。如果是多台服务器指标，那么可以根据标签拆分，以集群的维度查看指标

### topk()

取前几位的最高值，gauge或者counter类型的数据都可以，但是如果是counter类型，需要在内部increase()或者rate()，一般用来做瞬时报警

```mathematica
topk(1,count_netstat_wait_connections)
```

### count()

把数值符合条件的，输出数目进行加和

```mathematica
count(count_netstat_wait_connections>10)
```

### more

https://prometheus.io/docs/prometheus/latest/querying/functions/

## cpu usage

+ 用户态user
+ 内核态sys
+ 空闲idle(cpu啥都没有干)
+ 其它几个状态的cpu使用

![在这里插入图片描述](https://img-blog.csdnimg.cn/b67a7b7807074314901da6c43014d7b3.png)


```mathematica
(1-(sum(increase(node_cpu_seconds_total{mode="idle"}[1m])) by (instance)/sum(increase(node_cpu_seconds_total[1m])) by (instance)))*100 
```

## exporter

### python

```python
@app.route('/metrics')
def metrics():
    g_monitor = Monitor()
    g_monitor.set_prometheus()
    # 通过Metrics接口返回统计结果
    registry = g_monitor.get_prometheus_metrics_info()
    return Response(generate_latest(registry), mimetype="text/plain")

```

```python
class Monitor:
    def __init__(self):
        self.collector_registry = CollectorRegistry(auto_describe=True)

        self.summary = Summary(name="summary_test",
                               documentation="test in summary",
                               labelnames=("test",),
                               registry=self.collector_registry)

        self.gauge = Gauge(name="gauge_test",
                           documentation="test in gauge",
                           labelnames=("test",),
                           registry=self.collector_registry)

        self.counter = Counter(name="counter_test",
                               documentation="test in counter",
                               labelnames=("test",),
                               registry=self.collector_registry)

        self.histogram = Histogram(name="histogram_test",
                                   documentation="test in histogram",
                                   labelnames=("test",),
                                   registry=self.collector_registry)
        ProcessCollector(namespace="pc", pid=lambda: os.getpid(), registry=self.collector_registry)

    # 获取/metrics结果
    def get_prometheus_metrics_info(self):
        return self.collector_registry

    def set_prometheus(self):
        self.summary.labels("haha").observe(10)
        self.gauge.labels("hehe").set(1)
        self.counter.labels("zeze").inc()
        self.histogram.labels("tete").observe(random.randint(-10, 10))

```

### golang

```golang
func recordMetrics() {
	go func() {
		for {
			opsProcessed.Inc()
			time.Sleep(2 * time.Second)
		}
	}()
}

var (
	opsProcessed = promauto.NewCounter(prometheus.CounterOpts{
		Name: "myapp_processed_ops_total",
		Help: "The total number of processed events",
	})
)

func main() {
	foo := metrics.NewFooCollector()
	prometheus.MustRegister(foo)
	recordMetrics()

	http.Handle("/metrics", promhttp.Handler())
	http.ListenAndServe(":2112", nil)
}
```

```go
type fooCollector struct {
	fooMetric *prometheus.Desc
	barMetric *prometheus.Desc
}

func NewFooCollector() *fooCollector {
	c := &fooCollector{
		fooMetric: prometheus.NewDesc("foo_metric",
			"Shows whether a foo has occurred in our cluster",
			nil, map[string]string{"type": "foo"},
		),
		barMetric: prometheus.NewDesc("bar_metric",
			"Shows whether a bar has occurred in our cluster",
			nil, map[string]string{"type": "bar"},
		),
	}
	return c
}

func (collector *fooCollector) Describe(ch chan<- *prometheus.Desc) {
	ch <- collector.fooMetric
	ch <- collector.barMetric
}

func (collector *fooCollector) Collect(ch chan<- prometheus.Metric) {
	rand.Seed(time.Now().Unix())
	ch <- prometheus.MustNewConstMetric(collector.fooMetric, prometheus.GaugeValue, rand.Float64())
	ch <- prometheus.MustNewConstMetric(collector.barMetric, prometheus.CounterValue, rand.Float64())
}
```

## prometheus-operator

```shell
helm search repo prometheus-opeeator
helm install prometheus-operator stable/prometheus-operator
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e29f06c9791c4ca0adfd0305384e528a.png)


## 参考文档

https://prometheus.io/docs/introduction/overview/

https://blog.csdn.net/qq_22227087/article/details/100698791

https://www.cnblogs.com/wgx519/p/13951377.html

