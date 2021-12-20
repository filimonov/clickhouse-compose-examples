# 3 nodes cluster

## Usage:

### Latest version

```
docker-compose up -d

$ docker-compose exec clickhouse1 bash
root@clickhouse1:/# clickhouse-client 
ClickHouse client version 21.11.5.33 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 21.11.5 revision 54450.

clickhouse1 :) select hostName(), * from clusterAllReplicas('{cluster}',system.one) format PrettyCompactMonoBlock;

SELECT
    hostName(),
    *
FROM clusterAllReplicas('{cluster}', system.one)
FORMAT PrettyCompactMonoBlock

Query id: f2b1aeb5-5e93-4605-b550-9259a5618913

┌─hostName()──┬─dummy─┐
│ clickhouse1 │     0 │
│ clickhouse3 │     0 │
│ clickhouse2 │     0 │
└─────────────┴───────┘

3 rows in set. Elapsed: 0.011 sec. 

exit;

docker-compose down
```

### Some other version / docker images


```
$ CLICKHOUSE_SERVER_IMAGE=yandex/clickhouse-server:21.3 docker-compose up -d
Creating network "3_nodes_1x3_default" with the default driver
Creating 3_nodes_1x3_zookeeper_1 ... done
Creating 3_nodes_1x3_clickhouse2_1 ... done
Creating 3_nodes_1x3_clickhouse3_1 ... done
Creating 3_nodes_1x3_clickhouse1_1 ... done
Creating 3_nodes_1x3_all_services_ready_1 ... done

$ docker-compose exec clickhouse1 bash
root@clickhouse1:/# clickhouse-client 
ClickHouse client version 21.3.18.4 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 21.3.18 revision 54447.

clickhouse1 :) 
```

## How to

### Mimic missing/offline replica

```
$docker-compose stop clickhouse3
Stopping 3_nodes_1x3_clickhouse3_1 ... done

$ docker-compose exec clickhouse1 bash
root@clickhouse1:/# clickhouse-client 
ClickHouse client version 21.11.5.33 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 21.11.5 revision 54450.


clickhouse1 :) select hostName(), * from clusterAllReplicas('{cluster}',system.one) format PrettyCompactMonoBlock;

SELECT
    hostName(),
    *
FROM clusterAllReplicas('{cluster}', system.one)
FORMAT PrettyCompactMonoBlock

Query id: 523d0b79-cede-4e13-8577-41ef35afbf80

↓ Progress: 2.00 rows, 2.00 B (18.39 rows/s., 18.39 B/s.) 
2 rows in set. Elapsed: 0.333 sec. 

Received exception from server (version 21.11.5):
Code: 279. DB::Exception: Received from localhost:9000. DB::Exception: All connection tries failed. Log: 

Code: 32. DB::Exception: Attempt to read after eof. (ATTEMPT_TO_READ_AFTER_EOF) (version 21.11.5.33 (official build))
Code: 209. DB::NetException: Timeout: connect timed out: 172.23.0.4:9000 (clickhouse3:9000, receive timeout 0 ms, send timeout 0 ms). (SOCKET_TIMEOUT) (version 21.11.5.33 (official build))
Code: 209. DB::NetException: Timeout: connect timed out: 172.23.0.4:9000 (clickhouse3:9000, receive timeout 0 ms, send timeout 0 ms). (SOCKET_TIMEOUT) (version 21.11.5.33 (official build))

: While executing Remote. (ALL_CONNECTION_TRIES_FAILED)

clickhouse1 :) set sk
skewPop                  skewPopIf                skewPopOrNull            skewSamp                 skewSampIf               skewSampOrNull           skip_unavailable_shards
skewPopArray             skewPopMerge             skewPopResample          skewSampArray            skewSampMerge            skewSampResample
skewPopDistinct          skewPopNull              skewPopSimpleState       skewSampDistinct         skewSampNull             skewSampSimpleState
skewPopForEach           skewPopOrDefault         skewPopState             skewSampForEach          skewSampOrDefault        skewSampState
clickhouse1 :) set skip_unavailable_shards=1;

SET skip_unavailable_shards = 1

Query id: 5fb485ef-37d2-46e1-a33e-bdfa8e406e24

Ok.

0 rows in set. Elapsed: 0.001 sec. 

clickhouse1 :) select hostName(), * from clusterAllReplicas('{cluster}',system.one) format PrettyCompactMonoBlock;

SELECT
    hostName(),
    *
FROM clusterAllReplicas('{cluster}', system.one)
FORMAT PrettyCompactMonoBlock

Query id: e39e767a-d83b-4602-b897-1524276442ef

┌─hostName()──┬─dummy─┐
│ clickhouse1 │     0 │
│ clickhouse2 │     0 │
└─────────────┴───────┘

2 rows in set. Elapsed: 0.701 sec. 

...
```
