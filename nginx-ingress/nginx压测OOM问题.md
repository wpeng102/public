# 问题现象
最近客户在K8s集群上做Nginx ingress压测，pod limit设置为32C 64G。在4w+ TPS压力下持续压测发现Pod 内存不断上涨，最后出现Nginx ingress Pod OOM。
通过pprof goroutine发现（go tool pprof -svg -output nginx-ingress-gorouting.svg  http://192.168.0.47:10254/debug/pprof/goroutine）goroutine协程数量不断上涨，
nginx进程有大量且持续不断增长的链接。

# 解决方式
最终需要通过关闭 enable-metrics进行规避。

# 根本原因
Nginx采用SUMMARIES方式采集普罗需要的metrics数据，该方式在压测情况下需要消耗大量client端计算资源。导致routine协程获取锁计算较慢，协程释放速度慢于生成速度。消耗大量内存资源。

# 链接
https://prometheus.io/docs/practices/histograms/
