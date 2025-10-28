==Overview Metrics==
1. Number of topics: Helps identify unbounded topic creation; too many can affect metadata load times.
2. Number of partitions: Drives controller load and Zookeeper/ KRaft metadata.
3. Offline partitions: Partition unavailable - can't accept reads/ writes (Always 0)
4. Under-replicated partitions: Replica lag or broker unavailable, risk of losing replicas.
```
0 = health
> 0 for > 1 min = Warning
> 10 for > 5 min = Critical
```
5. Active controller count: More or fewer controllers -> leadership, churn, reassignments. Must be 1
6. Cluster broker count: Detect missing brokers. Warning if < N

==Zookeeper/ KRaft Metrics==
1. 

==Broker JVM Metrics==
1. Heap used %: Memory pressure -> GC pauses, producer latency spikes.
```
> 75% = Warn
> 85% = Critical
```
2. GC pause time (99th %): Too frequent/ long GC pauses = producer timeouts.
```
> 500 ms = Warn
> 1 s = Critical
```
3. Thread count/ blocked threads: Thread leaks, stuck I/o
```
Trend up > 25% = Warn
```
4. Open file descriptors: 
```
> 80% of ulimit = Critical
```

==Broker Core Metrics==
1. Request queue size/ handler idle ratio: Backpressure in network threads
```
> 50 pending requests = Warn
```
2. Bytes in/out rate: Indicates load and network utilization.
```
Sudden drops > 80% = Alert
```
3. Replication bytes in/out: Replication saturation causes lag.
```
> 70% network capacity = Warn
```
4. Leader election rate: Frequent elections = controller churn
```
> 5 elections min = Warn
```

==Producer Metrics==
1. Producer write/s: Total message throughput.
```
Basline based; drop > 30% = Warn
```
2. Bytes out by topic: Detect topic skew/ hot partition.
```
> 80% traffic to single topic = Warn
```
3. Request rate/ latency: Broker write response time.
```
Avg > 200 ms = Warn; > 1 s = Critical
```
4. I/O wait time: Disk I/O bottleneck.
```
> 30% of CPU = Warn
```
5. Record error rate: Retries/ acks failures.
```
> 0.1% = Warn
> 1% = Critical
```

==Consumer Metrics==
1. Consumer read/s: Ingestion throughput.
```
Trend based
```
2. Consumer lag by group: Measures delay from producers.
```
> 1 min lag = Warn
> 5 min = Critical
```
3. Bytes consumed/ messages consumed: Throughput visibility
```
N/A - trend
```
4. Min fetch rate: Stuck consumers = 0 fetches/sec.
```
= 0 for > 2 min = Critical.
```

==Schema Registry Metrics==
1. Registry availability: Producers/ consumers can't deserialize messages if down.
```
= 100% uptime; < 100% = Critical
```
2. Number of schemas / versions: Growth indicates evolution and storage load.
```
Monitor growth trend
```

==Kafka Connect Metrics==
1. Connect cluster status: If down, no data flows in/out.
```
> 0 failed tasks = Warn;
> 5 = Critical
```
2. Task errors: Connector failures (blocked pipeline)
```
> 0 = Warn
```
3. Sink/source throughput: Bottleneck diagnosis.
```
Throughput drop > 40% = Warn
```

==Log/ Disk Metrics==
1. Broker log directory free space: Full disk = broker crash.
```
< 15% free = Critical
```
2. Log flush time: Disk latency
```
> 100 ms avg = Warn
```
3. Log directory failures: File system errors or I/O corruption.
```
> 0 = Critical
```

==Cluster Health and Infra==
1. Controller queue size: Controller overload causes delayed leadership elections.
```
> 10 queued tasks = Warn
```
2. Request handler idle percent: Low = thread starvation.
```
< 20% idle = Warn
```
3. Network processor idle percent: Low = network overloaded.
```
<20% idle = Warn
```
4. Disk I/O utilization: Saturation -> latency.
```
> 85% = Warn
> 95% = Critical
```

5. Broker health...CPU, memory
6. Total number of partitions
7. Number of consumers
8. Consumer wait time, Poll time, min. fetch bytes
9. 