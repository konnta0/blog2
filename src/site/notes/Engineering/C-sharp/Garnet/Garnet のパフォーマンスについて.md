---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"Garnet のパフォーマンスについて","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Garnet のパフォーマンスについて","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/C-sharp/Garnet/Garnet のパフォーマンスについて/","metatags":{"og:title":"Garnet のパフォーマンスについて","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Garnet のパフォーマンスについて","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-10-06T01:03:56.823+09:00","updated":"2025-10-07T02:57:39.822+09:00"}
---

#csharp #dontet #garnet  #redis #cache #benchmark


https://microsoft.github.io/garnet/docs/benchmarking/results-resp-bench

## TL;DR
Garnet は Write が非常に高速で、Read も Redis と比較しても大きな差はありませんでした。
またクライアント数が増えても安定してパフォーマンスが出ているような結果でした。
逆に Redis はクライアント数が増えると急激にパフォーマンスが落ちる傾向があります。
## 実際に計測してみた
ドキュメントを参考にローカルで Garnet と Redis のパフォーマンスを比較してみました。

### 環境
Apple: M2 Pro
macOS Sequoia: 15.4.1
memory: 32GB

### セットアップ
dotnet の実行環境は準備済みの前提です。

1: Garnet の [GitHub リポジトリ](https://github.com/microsoft/garnet)をクローン
```bash
git clone https://github.com/microsoft/garnet.git
```

Garnet を起動する場合
```bash
host=127.0.0.1
port=6379

dotnet run -c Release --framework=net9.0 --project main/GarnetServer -- \
--bind $host \
--port $port \
--no-pubsub \
--no-obj \
--index 1g


    _________
   /_||___||_\      Garnet 1.0.84 64 bit; standalone mode
   '. \   / .'      Listening on: 127.0.0.1:6379
     '.\ /.'        https://aka.ms/GetGarnet
       '.'

* Ready to accept connections

```

Redis を起動する場合(少し古いですが Redis 7.0.8 を使用)
```bash
./redis-server \
	--bind $host \
	--port $port \
	--logfile "" \
	--save "" \
	--appendonly no \
	--protected-mode no \
	--io-threads 32
65402:C 07 Oct 2025 01:22:15.071 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
65402:C 07 Oct 2025 01:22:15.072 # Redis version=7.0.8, bits=64, commit=00000000, modified=0, pid=65402, just started

```


### ベンチマーク

GET のパフォーマンスを計測する
```bash
host=127.0.0.1
port=6379
valuelength=128
batchsize=1
dbsize=16777216

dotnet run -c Release --framework=net9.0 --project benchmark/Resp.benchmark \
  --host $host \
  --port $port \
  --op GET \
  --keylength 8 \
  --valuelength $valuelength \
  --threads 1,2,4,8,16,32,64,128 \
  --batchsize $batchsize \
  --dbsize $dbsize
```

Garnet の実行結果
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Throughput
skipLoad: Disabled
DBsize: 16777216
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 15
NumThreads: 1,2,4,8,16,32,64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: LightClient
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------

Generating 4096 MSET request batches of size 4096 each; total 16777216 ops
Resizing request buffer from 65536 to 131072
Resizing request buffer from 131072 to 262144
Resizing request buffer from 262144 to 524288
Resizing request buffer from 524288 to 1048576
Request generation complete
maxBytesWritten out of maxBufferSize: 614417/1048576
Loading time: 20.611 secs

Operation type: MSET
Num threads: 8
Total time: 3,221.00ms for 16,777,216.00 ops
Throughput: 5,208,697.92 ops/sec
Restricting #buffers to 65536 instead of 33554432

Generating 65536 GET request batches of size 1 each; total 65536 ops
Request generation complete
maxBytesWritten out of maxBufferSize: 27/65536
Loading time: 0.836 secs

Operation type: GET
Num threads: 1
Total time: 15,005.00ms for 373,268.00 ops
Throughput: 24,876.24 ops/sec

Operation type: GET
Num threads: 2
Total time: 15,005.00ms for 458,033.00 ops
Throughput: 30,525.36 ops/sec

Operation type: GET
Num threads: 4
Total time: 15,005.00ms for 769,790.00 ops
Throughput: 51,302.23 ops/sec

Operation type: GET
Num threads: 8
Total time: 15,005.00ms for 1,074,386.00 ops
Throughput: 71,601.87 ops/sec

Operation type: GET
Num threads: 16
Total time: 15,007.00ms for 1,223,065.00 ops
Throughput: 81,499.63 ops/sec

Operation type: GET
Num threads: 32
Total time: 15,006.00ms for 1,385,600.00 ops
Throughput: 92,336.40 ops/sec

Operation type: GET
Num threads: 64
Total time: 15,005.00ms for 1,528,231.00 ops
Throughput: 101,848.12 ops/sec

Operation type: GET
Num threads: 128
Total time: 15,009.00ms for 1,723,916.00 ops
Throughput: 114,858.82 ops/sec
```

Redis の実行結果
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Throughput
skipLoad: Disabled
DBsize: 16777216
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 15
NumThreads: 1,2,4,8,16,32,64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: LightClient
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------

Generating 4096 MSET request batches of size 4096 each; total 16777216 ops
Resizing request buffer from 65536 to 131072
Resizing request buffer from 131072 to 262144
Resizing request buffer from 262144 to 524288
Resizing request buffer from 524288 to 1048576
Request generation complete
maxBytesWritten out of maxBufferSize: 614417/1048576
Loading time: 20.401 secs

Operation type: MSET
Num threads: 8
Total time: 19,187.00ms for 16,777,216.00 ops
Throughput: 874,405.38 ops/sec
Restricting #buffers to 65536 instead of 33554432

Generating 65536 GET request batches of size 1 each; total 65536 ops
Request generation complete
maxBytesWritten out of maxBufferSize: 27/65536
Loading time: 1.297 secs

Operation type: GET
Num threads: 1
Total time: 15,004.00ms for 387,965.00 ops
Throughput: 25,857.44 ops/sec

Operation type: GET
Num threads: 2
Total time: 15,003.00ms for 622,014.00 ops
Throughput: 41,459.31 ops/sec

Operation type: GET
Num threads: 4
Total time: 15,003.00ms for 950,833.00 ops
Throughput: 63,376.19 ops/sec

Operation type: GET
Num threads: 8
Total time: 15,006.00ms for 1,429,158.00 ops
Throughput: 95,239.10 ops/sec

Operation type: GET
Num threads: 16
Total time: 15,005.00ms for 1,631,821.00 ops
Throughput: 108,751.82 ops/sec

Operation type: GET
Num threads: 32
Total time: 15,005.00ms for 1,734,260.00 ops
Throughput: 115,578.81 ops/sec

Operation type: GET
Num threads: 64
Total time: 15,038.00ms for 1,872,499.00 ops
Throughput: 124,517.82 ops/sec

Operation type: GET
Num threads: 128
Total time: 15,126.00ms for 44,364.00 ops
Throughput: 2,932.96 ops/sec
```

Get の場合、Redis はスレッド数 128 で急激にパフォーマンスが落ちています。
![Pasted image 20251007013745.png](/img/user/Pasted%20image%2020251007013745.png)

余談でMSET の場合、Garnet は Redis の 5.9倍のパフォーマンスを発揮しています。
![Pasted image 20251007013817.png](/img/user/Pasted%20image%2020251007013817.png)

Read は Redis と比較しても大きな差はありませんが、Write はかなり高速なことがわかります。

次に SET を試してみます

```bash

host=127.0.0.1
port=6379
valuelength=128
batchsize=1
dbsize=16777216

dotnet run -c Release --framework=net9.0 --project benchmark/Resp.benchmark \
	--host $host \
	--port $port \
	--op SET \
	--keylength 8 \
	--valuelength $valuelength \
	--threads 1,2,4,8,16,32,64,128 \
	--batchsize $batchsize \
	--dbsize $dbsize
```

Garnet
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Throughput
skipLoad: Disabled
DBsize: 16777216
TotalOps: 33554432
Op Benchmark: SET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 15
NumThreads: 1,2,4,8,16,32,64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: LightClient
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------

Generating 4096 MSET request batches of size 4096 each; total 16777216 ops
Resizing request buffer from 65536 to 131072
Resizing request buffer from 131072 to 262144
Resizing request buffer from 262144 to 524288
Resizing request buffer from 524288 to 1048576
Request generation complete
maxBytesWritten out of maxBufferSize: 614417/1048576
Loading time: 20.617 secs

Operation type: MSET
Num threads: 8
Total time: 3,416.00ms for 16,777,216.00 ops
Throughput: 4,911,363.00 ops/sec
Restricting #buffers to 65536 instead of 33554432

Generating 65536 SET request batches of size 1 each; total 65536 ops
Request generation complete
maxBytesWritten out of maxBufferSize: 163/65536
Loading time: 0.835 secs

Operation type: SET
Num threads: 1
Total time: 15,030.00ms for 355,899.00 ops
Throughput: 23,679.24 ops/sec

Operation type: SET
Num threads: 2
Total time: 15,005.00ms for 458,448.00 ops
Throughput: 30,553.02 ops/sec

Operation type: SET
Num threads: 4
Total time: 15,005.00ms for 792,266.00 ops
Throughput: 52,800.13 ops/sec

Operation type: SET
Num threads: 8
Total time: 15,005.00ms for 1,042,501.00 ops
Throughput: 69,476.91 ops/sec

Operation type: SET
Num threads: 16
Total time: 15,002.00ms for 1,122,309.00 ops
Throughput: 74,810.63 ops/sec

Operation type: SET
Num threads: 32
Total time: 15,005.00ms for 1,478,596.00 ops
Throughput: 98,540.22 ops/sec

Operation type: SET
Num threads: 64
Total time: 15,006.00ms for 1,556,088.00 ops
Throughput: 103,697.72 ops/sec

Operation type: SET
Num threads: 128
Total time: 15,006.00ms for 1,768,729.00 ops
Throughput: 117,868.12 ops/sec
```

Redis
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Throughput
skipLoad: Disabled
DBsize: 16777216
TotalOps: 33554432
Op Benchmark: SET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 15
NumThreads: 1,2,4,8,16,32,64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: LightClient
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------

Generating 4096 MSET request batches of size 4096 each; total 16777216 ops
Resizing request buffer from 65536 to 131072
Resizing request buffer from 131072 to 262144
Resizing request buffer from 262144 to 524288
Resizing request buffer from 524288 to 1048576
Request generation complete
maxBytesWritten out of maxBufferSize: 614417/1048576
Loading time: 18.878 secs

Operation type: MSET
Num threads: 8
Total time: 18,685.00ms for 16,777,216.00 ops
Throughput: 897,897.56 ops/sec
Restricting #buffers to 65536 instead of 33554432

Generating 65536 SET request batches of size 1 each; total 65536 ops
Request generation complete
maxBytesWritten out of maxBufferSize: 163/65536
Loading time: 0.901 secs

Operation type: SET
Num threads: 1
Total time: 15,005.00ms for 384,230.00 ops
Throughput: 25,606.80 ops/sec

Operation type: SET
Num threads: 2
Total time: 15,005.00ms for 628,027.00 ops
Throughput: 41,854.52 ops/sec

Operation type: SET
Num threads: 4
Total time: 15,005.00ms for 850,736.00 ops
Throughput: 56,696.83 ops/sec

Operation type: SET
Num threads: 8
Total time: 15,005.00ms for 1,320,627.00 ops
Throughput: 88,012.46 ops/sec

Operation type: SET
Num threads: 16
Total time: 15,005.00ms for 1,425,479.00 ops
Throughput: 95,000.27 ops/sec

Operation type: SET
Num threads: 32
Total time: 15,002.00ms for 1,466,709.00 ops
Throughput: 97,767.56 ops/sec

Operation type: SET
Num threads: 64
Total time: 15,006.00ms for 1,387,583.00 ops
Throughput: 92,468.55 ops/sec

Operation type: SET
Num threads: 128
Total time: 15,068.00ms for 38,379.00 ops
Throughput: 2,547.05 ops/sec

```

SET の場合も Redis はスレッド数 128 で急激にスループットが落ちています。
逆に Garnet はスレッド数が増えても安定してパフォーマンスが出ています。
![Pasted image 20251007020140.png](/img/user/Pasted%20image%2020251007020140.png)


ローカルなのとこれで設定として等価的なものなのかあまり自信がないですが、規模が大きくなってもスループットが出しやすいのは Garnet なのか？という感触を得ました。


次にレイテンシーを計測してみます。GET:80, SET:20 の割合で実行してみます。

```bash
valuelength=128
batchsize=1

dotnet run -c Release --framework=net9.0 --project benchmark/Resp.benchmark \
-- \
--host $host \
--port $port \
--batchsize 1 \
--threads 1,2,4,8,16,32,64,128 \
--client GarnetClientSession \
--runtime 35 \
--op-workload GET,SET \
--op-percent 80,20 \
--online \
--valuelength $valuelength \
--keylength 8 \
--dbsize 1024 \
--itp $batchsize

```

Garnet
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Online
skipLoad: Disabled
DBsize: 1024
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 35
NumThreads: 1,2,4,8,16,32,64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: GarnetClientSession
Pool: False
OpWorkload GET,SET
OpPercent 80,20
Intra thread parallelism: 1
SyncMode: False
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------
Running benchmark using GarnetClientSession client type
Using OpRunnerGarnetClientSession...
[000.02:15:50.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.02:15:50.(info)] |online| 16.32          29.31          36.35          41.4           61.18          84.99          140.29         48084          48084          23.83
[000.02:15:52.(info)] |online| 16.32          29.31          36.35          40.8           59.65          81.92          135.17         98250          50166          25.02
[000.02:15:54.(info)] |online| 15.68          29.57          37.63          43.09          66.05          89.6           201.73         139365         41115          20.51
[000.02:15:56.(info)] |online| 15.68          29.57          37.89          42.81          64.26          87.04          168.96         186975         47610          23.75
[000.02:15:58.(info)] |online| 15.68          29.44          36.86          41.39          61.18          83.46          158.72         241571         54596          27.28
[000.02:16:00.(info)] |online| 15.68          29.57          37.38          42.19          62.72          83.97          153.6          284356         42785          21.34
[000.02:16:02.(info)] |online| 15.68          29.57          37.38          42.21          62.72          83.97          158.72         331510         47154          23.54
[000.02:16:04.(info)] |online| 15.68          29.57          37.12          41.53          61.18          82.94          154.62         385047         53537          26.7
[000.02:16:06.(info)] |online| 15.68          29.44          36.86          41.06          60.16          81.92          148.48         438148         53101          26.48
[000.02:16:08.(info)] |online| 15.68          29.44          36.35          40.61          59.14          80.38          143.36         492172         54024          26.94
[000.02:16:10.(info)] |online| 15.68          29.44          36.35          40.21          58.11          79.36          140.29         546755         54583          27.25
[000.02:16:12.(info)] |online| 15.36          29.44          36.1           40.1           57.86          78.85          139.26         598064         51309          25.59
[000.02:16:14.(info)] |online| 14.14          29.44          36.1           40.71          57.86          80.9           216.06         638014         39950          19.94
[000.02:16:16.(info)] |online| 14.14          29.44          36.1           40.88          58.11          82.94          268.29         684370         46356          23.18
[000.02:16:18.(info)] |online| 14.14          29.44          36.1           40.76          58.11          82.94          249.86         735472         51102          25.58
[000.02:16:20.(info)] |online| 14.14          29.44          35.84          40.58          57.6           82.43          239.62         788010         52538          26.27
[000.02:16:22.(info)] |online| 14.14          29.44          35.84          40.33          56.83          81.41          235.52         842365         54355          27.19
[000.02:16:24.(info)] |online| 14.14          29.44          35.84          40.27          56.58          80.9           229.38         893234         50869          25.43
[000.02:16:26.(info)] |online| 14.14          29.44          35.84          40.1           56.06          79.87          221.18         946987         53753          26.89
[000.02:16:28.(info)] |online| 14.14          29.44          35.84          40.04          56.06          79.87          216.06         998239         51252          25.63
[000.02:16:30.(info)] |online| 14.14          29.44          35.84          40             56.06          79.87          215.04         1049024        50785          25.44
[000.02:16:32.(info)] |online| 14.14          29.44          35.84          39.93          55.81          79.36          207.87         1100790        51766          25.88
[000.02:16:34.(info)] |online| 14.14          29.44          35.84          39.98          55.55          78.85          205.82         1149593        48803          24.4
[000.02:16:36.(info)] |online| 14.14          29.44          35.84          39.89          55.55          78.33          201.73         1202238        52645          26.34
[000.02:16:38.(info)] |online| 14.14          29.44          35.84          39.76          55.04          77.82          197.63         1256364        54126          27.12
[000.02:16:40.(info)] |online| 14.14          29.44          35.84          39.73          55.04          77.82          196.61         1307350        50986          25.49
[000.02:16:42.(info)] |online| 14.14          29.44          36.1           40.05          56.32          79.36          196.61         1346864        39514          19.76
[000.02:16:44.(info)] |online| 14.14          29.44          36.1           40.3           57.09          80.38          199.68         1388124        41260          20.64
[000.02:16:46.(info)] |online| 14.14          29.44          36.1           40.18          56.83          79.87          198.66         1442028        53904          26.95
[000.02:16:48.(info)] |online| 14.14          29.44          36.1           40.07          56.32          79.36          194.56         1495740        53712          26.86
[000.02:16:50.(info)] |online| 16.64          29.31          35.07          36.53          43.52          51.46          82.43          54700          54700          27.35
[000.02:16:52.(info)] |online| 15.36          29.18          35.33          37.56          46.08          59.9           129.54         106411         51711          25.86
[000.02:16:54.(info)] |online| 15.36          29.44          36.35          40.38          55.81          78.85          249.86         148309         41898          20.99
[000.02:16:56.(info)] |online| 15.36          29.44          35.84          39.34          52.74          74.24          197.63         203055         54746          27.37
[000.02:16:58.(info)] |online| 15.36          29.44          35.58          38.88          51.2           71.68          162.82         256821         53766          26.9


```

Redis
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Online
skipLoad: Disabled
DBsize: 1024
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 35
NumThreads: 1,2,4,8,16,32,64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: GarnetClientSession
Pool: False
OpWorkload GET,SET
OpPercent 80,20
Intra thread parallelism: 1
SyncMode: False
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------
Running benchmark using GarnetClientSession client type
Using OpRunnerGarnetClientSession...
[000.02:19:04.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.02:19:04.(info)] |online| 14.14          25.86          31.23          34.61          46.08          60.16          119.81         57557          57557          28.51
[000.02:19:06.(info)] |online| 14.14          25.09          30.46          33.1           43.01          55.55          96.77          121188         63631          31.74
[000.02:19:08.(info)] |online| 14.14          25.22          30.34          32.84          42.5           54.78          98.3           182973         61785          30.83
[000.02:19:10.(info)] |online| 14.14          25.09          30.34          32.8           42.24          54.78          103.94         244125         61152          30.5
[000.02:19:12.(info)] |online| 13.89          25.09          30.34          32.94          41.98          56.83          129.02         303640         59515          29.76
[000.02:19:14.(info)] |online| 13.31          25.09          30.46          33.57          44.54          61.44          171.01         357377         53737          26.81
[000.02:19:16.(info)] |online| 13.31          25.22          30.46          33.43          43.78          59.9           155.65         418547         61170          30.51
[000.02:19:18.(info)] |online| 13.31          24.96          30.46          33.22          43.26          58.62          149.5          481325         62778          31.31
[000.02:19:20.(info)] |online| 13.31          25.09          30.46          33.2           43.26          58.37          148.48         541938         60613          30.23
[000.02:19:22.(info)] |online| 13.31          25.09          30.46          33.44          43.78          61.18          155.65         597637         55699          27.85
[000.02:19:24.(info)] |online| 13.31          25.09          30.46          33.5           44.29          61.44          157.7          656001         58364          29.17
[000.02:19:26.(info)] |online| 13.31          25.22          30.46          33.33          43.78          60.16          147.46         719251         63250          31.55
[000.02:19:28.(info)] |online| 13.31          25.09          30.46          33.23          43.52          59.39          144.38         781492         62241          31.04
[000.02:19:30.(info)] |online| 13.31          25.09          30.46          33.22          43.52          59.39          141.31         841896         60404          30.13
[000.02:19:32.(info)] |online| 13.31          25.09          30.46          33.78          44.03          62.46          240.64         887024         45128          22.53
[000.02:19:34.(info)] |online| 13.31          25.09          30.59          33.88          44.54          63.74          238.59         943317         56293          28.1
[000.02:19:36.(info)] |online| 13.31          25.09          30.59          33.85          44.29          63.23          229.38         1003299        59982          29.92
[000.02:19:38.(info)] |online| 13.31          25.22          30.59          33.77          44.03          62.46          219.14         1064473        61174          30.56
[000.02:19:40.(info)] |online| 13.31          25.22          30.46          33.68          43.78          61.7           211.97         1126787        62314          31.09
[000.02:19:42.(info)] |online| 13.31          25.22          30.59          33.7           43.78          61.95          206.85         1185254        58467          29.16
[000.02:19:44.(info)] |online| 13.31          25.22          30.59          33.75          44.03          62.98          211.97         1242715        57461          28.66
[000.02:19:46.(info)] |online| 13.31          25.34          30.59          33.75          44.03          63.23          211.97         1301989        59274          29.56
[000.02:19:48.(info)] |online| 13.31          25.34          30.59          33.72          44.03          62.72          205.82         1362392        60403          30.13
[000.02:19:50.(info)] |online| 13.31          25.34          30.59          33.67          43.78          62.46          200.7          1423491        61099          30.49
[000.02:19:52.(info)] |online| 13.31          25.34          30.59          33.63          43.78          61.95          199.68         1484856        61365          30.62
[000.02:19:54.(info)] |online| 13.31          25.34          30.59          33.69          44.03          62.72          201.73         1541067        56211          28.09
[000.02:19:56.(info)] |online| 13.31          25.47          30.59          33.73          44.03          62.46          199.68         1598422        57355          28.61
[000.02:19:58.(info)] |online| 13.31          25.47          30.59          33.71          44.03          62.46          195.58         1658610        60188          30.02
[000.02:20:00.(info)] |online| 13.31          25.47          30.59          33.69          44.03          61.95          189.44         1719017        60407          30.14
[000.02:20:02.(info)] |online| 13.31          25.47          30.59          33.64          43.78          61.44          185.34         1780850        61833          30.85
[000.02:20:04.(info)] |online| 14.4           25.6           30.46          32.92          43.01          58.37          131.07         60554          60554          30.28
[000.02:20:06.(info)] |online| 14.4           25.6           30.46          33.21          42.75          58.11          139.26         120180         59626          29.74
[000.02:20:09.(info)] |online| 14.4           25.6           30.46          33.25          42.75          59.9           174.08         180026         59846          29.89
[000.02:20:11.(info)] |online| 14.4           25.6           30.46          33.25          43.26          60.16          175.1          240019         59993          29.95
[000.02:20:13.(info)] |online| 14.4           25.73          30.46          33.15          42.5           58.37          163.84         300949         60930          30.4

```

レイテンシーでは Redis の方が若干良い結果となりました。
![Pasted image 20251007022351.png](/img/user/Pasted%20image%2020251007022351.png)

![Pasted image 20251007022445.png](/img/user/Pasted%20image%2020251007022445.png)

ここまでやっていて感じたのですが、Redis は単一スレッド(クライアント)だとスコアが良いのですが、スレッド数を増やすと急激にパフォーマンスが落ちる傾向があります。

その結果がレイテンシーにも影響がでていると考え、スレッド数を 64, 128 に制限して再度レイテンシーを測定してみました。

Garnet
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Online
skipLoad: Disabled
DBsize: 1024
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 35
NumThreads: 64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: GarnetClientSession
Pool: False
OpWorkload GET,SET
OpPercent 80,20
Intra thread parallelism: 1
SyncMode: False
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------
Running benchmark using GarnetClientSession client type
Using OpRunnerGarnetClientSession...
[000.02:27:59.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.02:27:59.(info)] |online| 53.76          266.24         458.75         496.43         745.47         1351.68        3915.78        257978         257978         127.65
[000.02:28:01.(info)] |online| 39.17          266.24         458.75         497.89         745.47         1327.1         4751.36        517570         259592         129.41
[000.02:28:03.(info)] |online| 39.17          270.33         460.8          495.62         733.18         1261.57        4390.91        779194         261624         130.42
[000.02:28:05.(info)] |online| 39.17          272.38         460.8          496.83         741.38         1261.57        4554.75        1035565        256371         127.87
[000.02:28:07.(info)] |online| 39.17          274.43         458.75         494.98         741.38         1245.18        4456.45        1298473        262908         131.26
[000.02:28:09.(info)] |online| 39.17          274.43         458.75         493.87         741.38         1220.61        4489.22        1560941        262468         130.97
[000.02:28:11.(info)] |online| 39.17          274.43         456.7          493.73         741.38         1204.22        4620.29        1821081        260140         129.75
[000.02:28:13.(info)] |online| 39.17          274.43         456.7          494.25         745.47         1228.8         4816.9         2078668        257587         128.54
[000.02:28:15.(info)] |online| 39.17          274.43         458.75         495.69         749.57         1236.99        4915.2         2331298        252630         126.06
[000.02:28:17.(info)] |online| 39.17          274.43         456.7          496.12         753.66         1236.99        4980.73        2587576        256278         127.88
[000.02:28:19.(info)] |online| 39.17          274.43         456.7          495.31         749.57         1228.8         4915.2         2850811        263235         131.35
[000.02:28:21.(info)] |online| 25.47          274.43         456.7          496.76         753.66         1253.38        5439.49        3100337        249526         124.58
[000.02:28:23.(info)] |online| 25.47          274.43         458.75         508.34         770.05         1449.98        7471.1         3281996        181659         90.6
[000.02:28:25.(info)] |online| 25.47          272.38         458.75         508.07         770.05         1458.18        7372.8         3536060        254064         126.72
[000.02:28:27.(info)] |online| 25.47          272.38         458.75         508.61         774.14         1466.37        7176.19        3784529        248469         123.92
[000.02:28:29.(info)] |online| 25.47          274.43         458.75         508.42         778.24         1458.18        7012.35        4038215        253686         126.59
[000.02:28:31.(info)] |online| 25.47          274.43         458.75         507.5          774.14         1441.79        6881.28        4298258        260043         129.7
[000.02:28:33.(info)] |online| 25.47          274.43         458.75         506.53         774.14         1425.41        6815.74        4559460        261202         130.41
[000.02:28:35.(info)] |online| 25.47          274.43         458.75         505.38         770.05         1409.02        6651.9         4823073        263613         131.74
[000.02:28:37.(info)] |online| 25.47          274.43         458.75         504.88         770.05         1392.64        6553.6         5081514        258441         129.09
[000.02:28:39.(info)] |online| 25.47          276.48         458.75         504.13         765.95         1368.06        6488.06        5343508        261994         130.67
[000.02:28:41.(info)] |online| 23.81          274.43         458.75         505.86         774.14         1400.83        6488.06        5578714        235206         117.31
[000.02:28:43.(info)] |online| 23.81          274.43         458.75         505.69         774.14         1400.83        6520.83        5833833        255119         127.43
[000.02:28:45.(info)] |online| 23.81          274.43         458.75         506.53         778.24         1409.02        6488.06        6077062        243229         121.43
[000.02:28:47.(info)] |online| 23.81          274.43         458.75         506.92         778.24         1409.02        6586.37        6324992        247930         123.84
[000.02:28:49.(info)] |online| 23.81          274.43         458.75         506.46         778.24         1400.83        6488.06        6583621        258629         128.29
[000.02:28:51.(info)] |online| 23.81          274.43         458.75         505.86         778.24         1384.45        6422.53        6848963        265342         133.54
[000.02:28:53.(info)] |online| 23.81          272.38         458.75         505.54         774.14         1368.06        6356.99        7106945        257982         129.83
[000.02:28:55.(info)] |online| 23.81          272.38         458.75         505.41         774.14         1368.06        6324.22        7361996        255051         128.68
[000.02:28:57.(info)] |online| 23.81          272.38         458.75         506.19         778.24         1384.45        6389.76        7603975        241979         121.78
[000.02:28:59.(info)] |online| 44.54          261.12         454.66         498.41         786.43         1294.34        6324.22        257329         257329         129.51
[000.02:29:01.(info)] |online| 44.54          262.14         456.7          502.16         786.43         1318.91        6356.99        510701         253372         127.51
[000.02:29:03.(info)] |online| 44.54          264.19         456.7          504.08         794.62         1351.68        6029.31        763199         252498         127.07
[000.02:29:05.(info)] |online| 44.54          264.19         458.75         504.58         798.72         1359.87        5931.01        1016591        253392         127.52
[000.02:29:07.(info)] |online| 44.54          268.29         458.75         503.38         790.53         1327.1         5668.86        1273791        257200         129.44

```

Redis
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Online
skipLoad: Disabled
DBsize: 1024
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 35
NumThreads: 64,128
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: GarnetClientSession
Pool: False
OpWorkload GET,SET
OpPercent 80,20
Intra thread parallelism: 1
SyncMode: False
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------
Running benchmark using GarnetClientSession client type
Using OpRunnerGarnetClientSession...
[000.02:30:05.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.02:30:05.(info)] |online| 116.22         331.78         667.65         3640.01        8519.68        105906.18      213909.5       35100          35100          16.96
[000.02:30:07.(info)] |online| 108.54         337.92         688.13         5009.76        14942.21       130547.71      236978.18      51621          16521          8.23
[000.02:30:09.(info)] |online| 108.54         352.26         864.26         5459.23        20054.02       137363.45      234881.02      71379          19758          9.84
[000.02:30:11.(info)] |online| 108.54         354.3          794.62         5633.64        20447.23       154140.67      251658.24      91340          19961          9.95
[000.02:30:13.(info)] |online| 91.65          360.45         876.54         5679.65        20840.45       146800.64      270532.61      113808         22468          11.23
[000.02:30:15.(info)] |online| 91.65          360.45         794.62         5367.31        19922.94       133169.15      254803.97      142926         29118          14.52
[000.02:30:17.(info)] |online| 63.23          358.4          667.65         4948.8         13631.49       130023.42      254803.97      182208         39282          19.6
[000.02:30:19.(info)] |online| 63.23          360.45         708.61         5242.79        18481.15       133169.15      255852.54      196049         13841          6.91
[000.02:30:21.(info)] |online| 63.23          360.45         667.65         4860.75        12320.77       128450.56      246415.36      237516         41467          20.69
[000.02:30:23.(info)] |online| 63.23          362.5          671.74         4805.1         12386.3        126877.7       241172.48      268048         30532          15.24
[000.02:30:25.(info)] |online| 63.23          364.54         700.42         4949.77        13631.49       130547.71      240123.9       285990         17942          8.95
[000.02:30:27.(info)] |online| 63.23          366.59         733.18         5154.2         16515.07       132120.58      241172.48      299483         13493          6.75
[000.02:30:29.(info)] |online| 63.23          368.64         765.95         5248.06        17694.72       132644.86      244318.21      318265         18782          9.36
[000.02:30:31.(info)] |online| 63.23          368.64         761.86         5145.16        16646.14       131072         240123.9       348417         30152          15.06
[000.02:30:33.(info)] |online| 63.23          370.69         811.01         5088.12        16711.68       128974.85      236978.18      378605         30188          15.05
[000.02:30:35.(info)] |online| 63.23          372.74         839.68         5165.13        17563.65       130547.71      236978.18      398387         19782          9.88
[000.02:30:37.(info)] |online| 63.23          374.78         851.97         5175.21        17694.72       131596.29      234881.02      421424         23037          11.5
[000.02:30:39.(info)] |online| 63.23          372.74         831.49         4986.08        14942.21       127926.27      232783.87      464159         42735          21.31
[000.02:30:41.(info)] |online| 63.23          372.74         786.43         4856.78        13500.42       126353.41      230686.72      502940         38781          19.34
[000.02:30:43.(info)] |online| 63.23          372.74         778.24         4732.23        11796.48       123207.68      228589.57      542850         39910          19.81
[000.02:30:45.(info)] |online| 63.23          372.74         765.95         4691.68        11796.48       122159.1       227540.99      575444         32594          16.29
[000.02:30:47.(info)] |online| 63.23          372.74         757.76         4716.03        13172.74       121634.82      226492.42      599683         24239          12.09
[000.02:30:49.(info)] |online| 63.23          374.78         782.34         4720.64        13697.02       121110.53      224395.26      625682         25999          12.99
[000.02:30:51.(info)] |online| 63.23          374.78         774.14         4733.24        14090.24       121110.53      223346.69      651168         25486          12.72
[000.02:30:53.(info)] |online| 63.23          372.74         753.66         4731.74        14221.31       120586.24      223346.69      677306         26138          13.05
[000.02:30:55.(info)] |online| 63.23          372.74         720.9          4637.97        13238.27       119537.66      221249.54      720238         42932          21.42
[000.02:30:57.(info)] |online| 63.23          372.74         733.18         4731.8         14614.53       120586.24      220200.96      733204         12966          6.47
[000.02:30:59.(info)] |online| 63.23          374.78         745.47         4734.17        15138.82       120061.95      220200.96      759268         26064          13.01
[000.02:31:01.(info)] |online| 63.23          374.78         733.18         4736.73        15269.89       120061.95      218103.81      786619         27351          13.65
[000.02:31:03.(info)] |online| 48.9           374.78         729.09         4693.09        15138.82       119013.38      216006.66      820728         34109          17.01
[000.02:31:05.(info)] |online| 83.97          370.69         557.05         2830.81        2277.38        76021.76       130023.42      44717          44717          22.2
[000.02:31:07.(info)] |online| 83.97          352.26         544.77         2737.71        2195.46        74448.9        137363.45      93147          48430          24.15
[000.02:31:09.(info)] |online| 83.97          360.45         552.96         2846.72        2146.3         76546.05       152043.52      135143         41996          20.96
[000.02:31:11.(info)] |online| 83.97          364.54         569.34         3309.95        5472.26        87031.81       156237.82      155266         20123          10.04
[000.02:31:13.(info)] |online| 58.11          366.59         585.73         3464.43        7012.35        90701.82       170917.89      186061         30795          15.37

```

128 並列だと Redis が耐えられなさそうでした。
![Pasted image 20251007023408.png](/img/user/Pasted%20image%2020251007023408.png)

64 並列(なら Redis が耐えられる)、かつ GET, SET の割合を逆転してみました。
```bash
valuelength=128
batchsize=1

dotnet run -c Release --framework=net9.0 --project benchmark/Resp.benchmark \
-- \
--host $host \
--port $port \
--batchsize 1 \
--threads 64 \
--client GarnetClientSession \
--runtime 35 \
--op-workload GET,SET \
--op-percent 20,80 \
--online \
--valuelength $valuelength \
--keylength 8 \
--dbsize 1024 \
--itp $batchsize
```

Garnet
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Online
skipLoad: Disabled
DBsize: 1024
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 35
NumThreads: 64
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: GarnetClientSession
Pool: False
OpWorkload GET,SET
OpPercent 20,80
Intra thread parallelism: 1
SyncMode: False
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------
Running benchmark using GarnetClientSession client type
Using OpRunnerGarnetClientSession...
[000.02:36:26.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.02:36:26.(info)] |online| 39.17          256            462.85         523.25         839.68         1712.13        8650.75        244736         244736         121.1
[000.02:36:28.(info)] |online| 39.17          262.14         462.85         509.41         790.53         1417.22        6815.74        506009         261273         130.18
[000.02:36:30.(info)] |online| 39.17          268.29         456.7          501.47         774.14         1302.53        6684.67        769990         263981         131.66
[000.02:36:32.(info)] |online| 39.17          268.29         456.7          501.94         778.24         1327.1         5963.78        1025003        255013         127.19
[000.02:36:34.(info)] |online| 39.17          268.29         454.66         498.26         765.95         1277.95        6127.62        1289697        264694         132.21
[000.02:36:36.(info)] |online| 39.17          272.38         454.66         497.92         765.95         1277.95        6029.31        1547765        258068         128.91
[000.02:36:38.(info)] |online| 39.17          270.33         454.66         506.76         786.43         1409.02        7700.48        1773837        226072         112.75
[000.02:36:40.(info)] |online| 36.1           272.38         456.7          507.05         786.43         1400.83        7634.94        2025496        251659         125.64
[000.02:36:42.(info)] |online| 36.1           272.38         456.7          508.44         790.53         1433.6         7405.57        2272163        246667         123.03
[000.02:36:44.(info)] |online| 36.1           270.33         456.7          507.29         786.43         1417.22        7274.5         2530273        258110         128.8
[000.02:36:46.(info)] |online| 36.1           270.33         456.7          508.45         790.53         1441.79        7274.5         2776731        246458         122.98
[000.02:36:48.(info)] |online| 36.1           270.33         458.75         507.82         786.43         1417.22        7143.42        3032323        255592         127.8
[000.02:36:50.(info)] |online| 36.1           272.38         458.75         507.32         786.43         1392.64        7143.42        3288201        255878         127.68
[000.02:36:52.(info)] |online| 36.1           272.38         458.75         506.72         782.34         1384.45        6881.28        3544763        256562         128.28
[000.02:36:54.(info)] |online| 36.1           272.38         458.75         506.39         786.43         1384.45        6815.74        3800165        255402         127.51
[000.02:36:56.(info)] |online| 36.1           272.38         458.75         505.2          782.34         1368.06        6684.67        4062873        262708         131.09
[000.02:36:58.(info)] |online| 36.1           270.33         458.75         505.73         786.43         1376.26        6619.14        4311585        248712         124.42
[000.02:37:00.(info)] |online| 36.1           270.33         458.75         505.01         782.34         1368.06        6520.83        4571726        260141         129.81
[000.02:37:02.(info)] |online| 25.09          270.33         458.75         504.11         782.34         1359.87        6520.83        4834212        262486         131.05
[000.02:37:04.(info)] |online| 25.09          270.33         458.75         504.6          782.34         1368.06        6488.06        5083556        249344         124.49
[000.02:37:06.(info)] |online| 25.09          270.33         458.75         505.01         786.43         1368.06        6520.83        5333063        249507         124.69
[000.02:37:08.(info)] |online| 25.09          270.33         458.75         504.72         782.34         1359.87        6488.06        5590033        256970         128.29
[000.02:37:10.(info)] |online| 25.09          270.33         458.75         503.56         778.24         1343.49        6389.76        5857553        267520         133.56
[000.02:37:12.(info)] |online| 25.09          270.33         458.75         504            782.34         1351.68        6389.76        6106776        249223         124.36
[000.02:37:14.(info)] |online| 25.09          270.33         458.75         504.21         782.34         1359.87        6455.3         6358179        251403         125.7
[000.02:37:16.(info)] |online| 25.09          270.33         458.75         504.42         782.34         1359.87        6455.3         6608926        250747         125.06
[000.02:37:18.(info)] |online| 25.09          270.33         458.75         503.19         778.24         1343.49        6324.22        6880940        272014         135.8
[000.02:37:20.(info)] |online| 25.09          270.33         458.75         508.11         786.43         1433.6         6946.82        7066624        185684         92.66
[000.02:37:22.(info)] |online| 25.09          270.33         458.75         507.98         786.43         1433.6         7045.12        7320892        254268         126.94
[000.02:37:24.(info)] |online| 25.09          270.33         458.75         507.72         786.43         1417.22        7012.35        7577189        256297         127.96
[000.02:37:26.(info)] |online| 45.06          270.33         456.7          511.08         819.2          1400.83        5799.94        250733         250733         125.3
[000.02:37:28.(info)] |online| 36.35          268.29         456.7          511.96         815.1          1515.52        6619.14        500318         249585         124.67
[000.02:37:30.(info)] |online| 36.35          272.38         458.75         508.87         798.72         1433.6         6815.74        755470         255152         127.38
[000.02:37:32.(info)] |online| 36.35          272.38         458.75         505.74         786.43         1359.87        6684.67        1013727        258257         128.87
[000.02:37:34.(info)] |online| 36.35          270.33         456.7          504.83         786.43         1351.68        7176.19        1269136        255409         127.7



```

Redis
```bash
<<<<<<< Benchmark Configuration >>>>>>>>
benchmarkType: Online
skipLoad: Disabled
DBsize: 1024
TotalOps: 33554432
Op Benchmark: GET
KeyLength: 8
ValueLength: 128
BatchSize: 1
RunTime: 35
NumThreads: 64
Auth:
Load Using SET: Disabled
ClientType used for benchmarking: GarnetClientSession
Pool: False
OpWorkload GET,SET
OpPercent 20,80
Intra thread parallelism: 1
SyncMode: False
TLS: Disabled
ClientHistogram: False
minWorkerThreads: 1000
minCompletionPortThreads: 1000
----------------------------------
Running benchmark using GarnetClientSession client type
Using OpRunnerGarnetClientSession...
[000.02:38:51.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.02:38:51.(info)] |online| 70.14          411.65         905.22         4660.43        22675.46       102236.16      193986.56      27168          27168          13.44
[000.02:38:53.(info)] |online| 70.14          389.12         720.9          3969.11        15794.18       92274.69       181403.65      64600          37432          18.61
[000.02:38:55.(info)] |online| 70.14          389.12         737.28         4346.8         17432.58       102236.16      181403.65      88641          24041          11.87
[000.02:38:57.(info)] |online| 70.14          380.93         684.03         4102.14        15925.25       96993.28       187695.1       125405         36764          18.36
[000.02:38:59.(info)] |online| 70.14          380.93         659.46         3954.56        14483.46       96468.99       181403.65      160931         35526          17.74
[000.02:39:01.(info)] |online| 70.14          382.98         630.78         3702.14        11206.66       93847.55       176160.77      207590         46659          23.27
[000.02:39:03.(info)] |online| 70.14          387.07         659.46         3989.36        12976.13       98041.86       178257.92      223472         15882          7.92
[000.02:39:05.(info)] |online| 62.21          385.02         696.32         3960.16        13762.56       96468.99       179306.5       259759         36287          18.1
[000.02:39:07.(info)] |online| 62.21          382.98         667.65         3782.2         11730.94       93323.26       179306.5       305256         45497          22.69
[000.02:39:09.(info)] |online| 62.21          387.07         696.32         3939.35        14352.38       95944.7        179306.5       326250         20994          10.47
[000.02:39:11.(info)] |online| 62.21          389.12         729.09         4069.3         15663.1        96993.28       180355.07      346789         20539          10.25
[000.02:39:13.(info)] |online| 62.21          387.07         733.18         4097.03        15532.03       98041.86       179306.5       375644         28855          14.39
[000.02:39:15.(info)] |online| 62.21          387.07         753.66         4182.5         17039.36       100139.01      179306.5       399676         24032          12.01
[000.02:39:17.(info)] |online| 62.21          389.12         753.66         4243.63        17956.86       100663.3       179306.5       423681         24005          11.98
[000.02:39:19.(info)] |online| 62.21          389.12         782.34         4386.49        19529.73       103809.02      180355.07      438961         15280          7.63
[000.02:39:21.(info)] |online| 62.21          391.17         765.95         4297.37        18087.94       102236.16      180355.07      478104         39143          19.52
[000.02:39:23.(info)] |online| 62.21          389.12         757.76         4195.29        16580.61       101187.58      179306.5       520380         42276          21.11
[000.02:39:25.(info)] |online| 62.21          391.17         770.05         4266.1         17432.58       102760.45      180355.07      542193         21813          10.88
[000.02:39:27.(info)] |online| 62.21          391.17         765.95         4282.46        17432.58       102236.16      181403.65      569264         27071          13.52
[000.02:39:29.(info)] |online| 62.21          391.17         761.86         4257.15        17301.5        102236.16      180355.07      603052         33788          16.89
[000.02:39:31.(info)] |online| 62.21          391.17         761.86         4267.7         16646.14       102760.45      182452.22      632107         29055          14.49
[000.02:39:33.(info)] |online| 62.21          391.17         761.86         4283.96        16777.22       103284.74      181403.65      659422         27315          13.58
[000.02:39:35.(info)] |online| 62.21          393.22         786.43         4283.92        17301.5        102236.16      181403.65      689038         29616          14.76
[000.02:39:37.(info)] |online| 62.21          391.17         765.95         4235.5         16515.07       101711.87      180355.07      727910         38872          19.39
[000.02:39:39.(info)] |online| 62.21          391.17         761.86         4228.7         16777.22       101187.58      180355.07      758797         30887          15.41
[000.02:39:41.(info)] |online| 62.21          391.17         770.05         4249.32        17301.5        101711.87      180355.07      785082         26285          13.12
[000.02:39:43.(info)] |online| 62.21          391.17         765.95         4232.34        17301.5        101187.58      179306.5       819420         34338          17.13
[000.02:39:45.(info)] |online| 62.21          391.17         753.66         4208.99        17170.43       101187.58      178257.92      853449         34029          16.97
[000.02:39:47.(info)] |online| 62.21          391.17         753.66         4227.97        17170.43       101711.87      179306.5       879735         26286          13.11
[000.02:39:49.(info)] |online| 62.21          391.17         765.95         4277.42        17563.65       102760.45      179306.5       900815         21080          10.51
[000.02:39:51.(info)] |online| 91.65          385.02         561.15         2921.49        1441.79        83886.08       164626.43      41873          41873          20.92
[000.02:39:53.(info)] |online| 56.83          378.88         573.44         3347.21        6127.62        85983.23       155189.25      75535          33662          16.79
[000.02:39:55.(info)] |online| 56.83          382.98         614.4          3495.57        6684.67        91226.11       159383.55      109698         34163          17.05
[000.02:39:57.(info)] |online| 56.83          385.02         643.07         3998.73        11337.73       105381.89      180355.07      128267         18569          9.18
[000.02:39:59.(info)] |online| 56.83          391.17         696.32         4614.34        22806.53       110624.77      183500.8       139278         11011          5.49


```

まぁ予想通りではあるのですが、 Redis の平均が悪いです。
![Pasted image 20251007024249.png](/img/user/Pasted%20image%2020251007024249.png)


## レイテンシーのまとめ
![Pasted image 20251007024830.png](/img/user/Pasted%20image%2020251007024830.png)

![Pasted image 20251007024850.png](/img/user/Pasted%20image%2020251007024850.png)

![Pasted image 20251007024904.png](/img/user/Pasted%20image%2020251007024904.png)

Redis はシングルスレッドで動いているので並列が増えるとレイテンシーが跳ね上がりました。
また、そこまで負荷の高くないワークロードであれば Garnet より高速な場合がありました。

逆に Garnet は並列に強いので大規模なワークロードには向いているような印象です。
また、書き込みが多くても安定しているように見えます。
