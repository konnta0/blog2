---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"Garnet のパフォーマンスについて","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Garnet のパフォーマンスについて","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/C-sharp/Garnet/Garnet のパフォーマンスについて/","metatags":{"og:title":"Garnet のパフォーマンスについて","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Garnet のパフォーマンスについて","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-10-07T03:21:01.000+09:00","updated":"2025-10-08T02:00:12.927+09:00"}
---

#csharp #dontet #garnet  #redis #cache #benchmark


https://microsoft.github.io/garnet/docs/benchmarking/results-resp-bench

# TL;DR
Garnet は Write が非常に高速で、Read も Redis と比較しても大きな差はありませんでした。
またクライアント数(並列数)が増えても安定してパフォーマンスが出ているような結果でした。

# 実際に計測してみた
ドキュメントを参考にローカルで Garnet と Redis のパフォーマンスを比較してみました。

### 環境
Apple: M2 Pro
macOS Sequoia: 15.4.1
memory: 32GB

## セットアップ
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


## ベンチマーク

### GET のパフォーマンスを計測する (実行結果詳細は ※1 を参照)
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

Get の場合、Redis はスレッド数 128 で急激にパフォーマンスが落ちています。
![Pasted image 20251007013745.png](/img/user/Pasted%20image%2020251007013745.png)

余談でデータ投入中の MSET で Garnet は Redis の **5.9倍**のパフォーマンスを発揮しています。
![Pasted image 20251007013817.png](/img/user/Pasted%20image%2020251007013817.png)

Read は Redis と比較しても大きな差はありませんが、Write はかなり高速なことがわかります。
### SET のパフォーマンスを計測する (実行結果詳細は ※2 を参照)
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

SET の場合も Redis はスレッド数 128 で急激にスループットが落ちています。
逆に Garnet はスレッド数が増えても安定してパフォーマンスが出ています。
![Pasted image 20251007020140.png](/img/user/Pasted%20image%2020251007020140.png)

ローカルなのとこれで設定として等価的なものなのかあまり自信がないですが、**規模が大きくなってもスループットが出しやすいのは Garnet** なのか？という感触を得ました。

### レイテンシーを計測する (実行結果詳細は ※3 を参照)
次にレイテンシーを計測してみます。
GET:80, SET:20 の割合で実行してみます。
Thread は 1 固定です。

```bash
valuelength=128
batchsize=1

dotnet run -c Release --framework=net9.0 --project benchmark/Resp.benchmark \
-- \
--host $host \
--port $port \
--batchsize 1 \
--threads 1 \
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

レイテンシーでは Redis の方が若干良い結果となりました。(誤差の範囲かもしれませんが)
![Pasted image 20251007022351.png](/img/user/Pasted%20image%2020251007022351.png)

Redis はスレッド数(クライアントの並列数)が少ないとスコアが良いのですが、スレッド数を増やすと**急激にパフォーマンスが落ちる**傾向があります。
逆に **Garnet はスレッド数が増えても安定**してパフォーマンスが出ています。

### スレッド数を調整して再度レイテンシーを計測する (実行結果詳細は ※4 を参照)
スループットを計測時には Redis は 128 並列で明らかな劣化が出ているように見えたので
スレッド数を 64 に制限して再度レイテンシーを測定してみました。

![Pasted image 20251008011102.png](/img/user/Pasted%20image%2020251008011102.png)
Redis が耐えられなさそうでした。
逆に Garnet は 64 並列でも安定して動作しています。
### スレッド数をさらに落として再度レイテンシーを計測する (実行結果詳細は ※5 を参照)
64 並列でも耐えられなさそうだったので、さらに並列数を落として 32 並列で試してみます。

![Pasted image 20251008003206.png](/img/user/Pasted%20image%2020251008003206.png)
Redis が安定して動作しました。おそらく手元の環境だと 32 並列が限界かもしれません。
### GET, SET の割合を逆転してみる (実行結果詳細は ※6 を参照)
64 並列、GET, SET の割合を逆転してみました。
```bash
valuelength=128
batchsize=1

dotnet run -c Release --framework=net9.0 --project benchmark/Resp.benchmark \
-- \
--host $host \
--port $port \
--batchsize 1 \
--threads 32 \
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

![Pasted image 20251008010137.png](/img/user/Pasted%20image%2020251008010137.png)
GET の比率が多い場合比較すると Redis のテイルレイテンシーが遅いような印象です。


### 余談: io-threads を指定して Redis を起動してみる

Redis は io-threads を指定して起動することができます。
```bash
./redis-server \
--bind $host \
--port $port \
--logfile "" \
--save "" \
--appendonly no \
--protected-mode no \
--io-threads 32
```

io-threads を指定して Redis を起動し、再度 32 並列で試してみました。

![Pasted image 20251008014552.png](/img/user/Pasted%20image%2020251008014552.png)
io-threads を指定した方が結果は良かったですが、Garnet には及びませんでした。
(注: このグラフは対数グラフです)

### 余談: Garnet で並列数を増やしてみる
ローカルなのであまり意味がないかもしれませんが、512 並列まで試しました。
グラフを見る限りは綺麗な推移を見せており急な性能劣化がないのでスケーリングの計画は立てやすそうです。
![Pasted image 20251008015747.png](/img/user/Pasted%20image%2020251008015747.png)
![Pasted image 20251008015759.png](/img/user/Pasted%20image%2020251008015759.png)

```bash

# 補足

### ※1 GET のパフォーマンス計測結果
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

### ※2 SET のパフォーマンス計測結果
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

### ※3 GET/SET 混在のパフォーマンス計測結果
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
NumThreads: 1
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
NumThreads: 1
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

### ※4 GET/SET 混在のパフォーマンス計測結果（スレッド数調整）
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
NumThreads: 64
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

### ※5 GET/SET 混在のパフォーマンス計測結果（スレッド数調整）
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
NumThreads: 32
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
[000.12:25:28.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.12:25:28.(info)] |online| 39.17          142.34         266.24         317.97         540.67         1155.07        6225.92        201415         201415         99.32
[000.12:25:30.(info)] |online| 39.17          151.55         256            290.24         464.9          864.26         4653.06        444747         243332         121.36
[000.12:25:32.(info)] |online| 22.4           154.62         256            295.12         454.66         1003.52        5242.88        654443         209696         104.38
[000.12:25:35.(info)] |online| 22.4           154.62         259.07         331.38         548.86         1843.2         8093.7         779723         125280         62.48
[000.12:25:37.(info)] |online| 22.4           154.62         258.05         319.58         507.9          1597.44        7503.87        1008867        229144         114.46
[000.12:25:39.(info)] |online| 22.4           154.62         258.05         323.01         520.19         1662.98        7798.78        1196419        187552         93.64
[000.12:25:41.(info)] |online| 22.4           154.62         257.02         315.29         497.66         1449.98        7274.5         1428704        232285         116.03
[000.12:25:43.(info)] |online| 22.4           155.65         257.02         308.1          473.09         1294.34        6782.98        1669732        241028         120.39
[000.12:25:45.(info)] |online| 22.4           156.67         256            302.28         454.66         1179.65        6324.22        1913794        244062         121.91
[000.12:25:47.(info)] |online| 22.4           157.7          256            297.81         442.37         1097.73        5996.54        2157365        243571         121.72
[000.12:25:49.(info)] |online| 22.4           157.7          254.98         294.43         434.18         1028.1         5734.4         2399448        242083         120.92
[000.12:25:51.(info)] |online| 22.4           157.7          254.98         291.76         425.98         966.66         5439.49        2641104        241656         120.53
[000.12:25:53.(info)] |online| 22.4           157.7          254.98         290.48         423.94         921.6          5308.42        2873274        232170         115.91
[000.12:25:55.(info)] |online| 22.4           157.7          254.98         291.06         425.98         925.7          5472.26        3087719        214445         106.96
[000.12:25:57.(info)] |online| 22.4           157.7          254.98         288.88         419.84         892.93         5210.11        3332684        244965         122.36
[000.12:25:59.(info)] |online| 22.4           158.72         254.98         287.17         415.74         860.16         5013.5         3575665        242981         121.25
[000.12:26:01.(info)] |online| 22.4           158.72         254.98         285.55         411.65         835.58         4849.66        3820335        244670         122.09
[000.12:26:03.(info)] |online| 22.02          159.74         254.98         284.66         407.55         815.1          4784.13        4057344        237009         118.33
[000.12:26:05.(info)] |online| 22.02          159.74         254.98         283.43         403.46         790.53         4685.82        4300998        243654         121.52
[000.12:26:07.(info)] |online| 22.02          160.77         254.98         282.47         401.41         770.05         4587.52        4542556        241558         120.48
[000.12:26:09.(info)] |online| 22.02          160.77         254.98         281.34         399.36         749.57         4521.98        4788446        245890         122.7
[000.12:26:11.(info)] |online| 22.02          160.77         253.95         280.25         397.31         729.09         4423.68        5035760        247314         123.35
[000.12:26:13.(info)] |online| 22.02          161.79         253.95         279.67         395.26         720.9          4390.91        5275230        239470         119.5
[000.12:26:15.(info)] |online| 22.02          161.79         253.95         278.51         391.17         700.42         4259.84        5527340        252110         125.8
[000.12:26:17.(info)] |online| 22.02          161.79         253.95         278            391.17         696.32         4145.15        5768108        240768         120.08
[000.12:26:19.(info)] |online| 22.02          161.79         253.95         277.83         391.17         692.22         4096           6002149        234041         116.85
[000.12:26:21.(info)] |online| 22.02          161.79         253.95         277.21         389.12         684.03         4014.08        6246463        244314         122.03
[000.12:26:23.(info)] |online| 22.02          161.79         253.95         276.59         387.07         671.74         3948.54        6492299        245836         122.61
[000.12:26:25.(info)] |online| 22.02          161.79         253.95         276.42         387.07         671.74         3932.16        6727989        235690         117.61
[000.12:26:27.(info)] |online| 22.02          162.82         253.95         276.09         387.07         663.55         3899.39        6967688        239699         119.85
[000.12:26:29.(info)] |online| 30.21          157.7          257.02         301.57         483.33         1179.65        5144.58        212416         212416         106
[000.12:26:31.(info)] |online| 30.21          162.82         252.93         280.7          411.65         880.64         3899.39        456501         244085         121.74
[000.12:26:33.(info)] |online| 21.12          164.86         251.9          272.61         382.98         733.18         3637.25        705070         248569         124.04
[000.12:26:35.(info)] |online| 21.12          165.89         252.93         269.79         374.78         659.46         3293.18        949978         244908         122.15
[000.12:26:37.(info)] |online| 21.12          165.89         251.9          268.35         370.69         622.59         3244.03        1193809        243831         121.67

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
NumThreads: 32
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
[000.12:27:06.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.12:27:06.(info)] |online| 23.17          143.36         248.83         314.12         593.92         933.89         6422.53        203784         203784         100.88
[000.12:27:08.(info)] |online| 23.17          151.55         248.83         297.21         552.96         819.2          4718.59        433252         229468         114.45
[000.12:27:10.(info)] |online| 23.17          153.6          247.81         289.81         540.67         782.34         3735.55        665639         232387         115.85
[000.12:27:12.(info)] |online| 23.17          154.62         246.78         283.99         532.48         765.95         3014.66        905214         239575         119.49
[000.12:27:14.(info)] |online| 23.17          155.65         246.78         283.84         528.38         757.76         2899.97        1131440        226226         112.83
[000.12:27:16.(info)] |online| 23.17          155.65         247.81         285.87         532.48         757.76         3325.95        1347036        215596         107.8
[000.12:27:18.(info)] |online| 23.17          156.67         248.83         288.79         536.58         774.14         3801.09        1555262        208226         103.91
[000.12:27:20.(info)] |online| 23.17          156.67         248.83         288.19         536.58         770.05         3915.78        1780935        225673         112.56
[000.12:27:22.(info)] |online| 23.17          156.67         248.83         288.16         536.58         770.05         3801.09        2003574        222639         111.04
[000.12:27:24.(info)] |online| 23.17          156.67         248.83         289            536.58         774.14         3768.32        2219412        215838         107.7
[000.12:27:26.(info)] |online| 23.17          156.67         248.83         287.29         536.58         765.95         3522.56        2455768        236356         117.88
[000.12:27:28.(info)] |online| 23.17          157.7          247.81         286.73         532.48         765.95         3342.34        2684085        228317         113.87
[000.12:27:30.(info)] |online| 23.17          157.7          247.81         285.85         532.48         761.86         3227.65        2916726        232641         116.03
[000.12:27:32.(info)] |online| 23.17          157.7          247.81         285.02         532.48         757.76         3342.34        3150156        233430         116.42
[000.12:27:34.(info)] |online| 22.78          156.67         246.78         283.85         528.38         753.66         3358.72        3388924        238768         119.15
[000.12:27:36.(info)] |online| 22.78          156.67         246.78         283.02         528.38         749.57         3260.42        3625207        236283         117.85
[000.12:27:38.(info)] |online| 22.78          156.67         246.78         281.77         524.29         745.47         3211.26        3868790        243583         121.49
[000.12:27:40.(info)] |online| 22.78          156.67         245.76         280.3          522.24         737.28         2998.27        4117797        249007         124.19
[000.12:27:42.(info)] |online| 20.86          156.67         245.76         279.85         520.19         733.18         2965.5         4353450        235653         117.53
[000.12:27:44.(info)] |online| 20.86          157.7          245.76         279.18         520.19         729.09         2883.58        4592483        239033         119.4
[000.12:27:46.(info)] |online| 20.86          157.7          245.76         278.37         518.14         724.99         2867.2         4836852        244369         121.88
[000.12:27:48.(info)] |online| 20.86          157.7          245.76         278.07         518.14         724.99         2867.2         5072639        235787         117.6
[000.12:27:50.(info)] |online| 20.86          157.7          245.76         278.12         518.14         724.99         2850.82        5301896        229257         114.46
[000.12:27:52.(info)] |online| 20.86          157.7          245.76         278.07         518.14         724.99         2818.05        5533364        231468         115.45
[000.12:27:54.(info)] |online| 20.86          157.7          245.76         278.49         518.14         724.99         2834.43        5755220        221856         110.71
[000.12:27:56.(info)] |online| 20.86          157.7          245.76         277.98         518.14         724.99         2752.51        5996176        240956         120.3
[000.12:27:58.(info)] |online| 20.86          157.7          245.76         277.98         518.14         724.99         2801.66        6226609        230433         114.93
[000.12:28:00.(info)] |online| 20.86          157.7          245.76         277.37         518.14         720.9          2801.66        6471422        244813         122.1
[000.12:28:02.(info)] |online| 20.86          157.7          244.74         276.9          516.1          720.9          2834.43        6713989        242567         120.98
[000.12:28:04.(info)] |online| 20.86          157.7          244.74         276.66         516.1          716.8          2768.9         6951293        237304         118.36
[000.12:28:06.(info)] |online| 35.58          156.67         244.74         272.88         516.1          679.94         1589.25        234862         234862         117.14
[000.12:28:08.(info)] |online| 35.58          157.7          243.71         270.14         505.86         663.55         2260.99        474466         239604         119.56
[000.12:28:10.(info)] |online| 17.79          157.7          245.76         274.14         512            704.51         2703.36        700908         226442         113.16
[000.12:28:12.(info)] |online| 17.79          157.7          244.74         273.18         509.95         692.22         2392.06        937490         236582         118.23
[000.12:28:14.(info)] |online| 17.79          157.7          243.71         270.16         501.76         671.74         2342.91        1185117        247627         123.5

```

### ※6  GET, SET の割合を逆転してみる
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
NumThreads: 32
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
[000.12:56:04.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.12:56:04.(info)] |online| 30.98          146.43         254.98         285.73         460.8          700.42         3915.78        223471         223471         110.85
[000.12:56:06.(info)] |online| 30.98          152.57         254.98         275.19         411.65         606.21         3653.63        467453         243982         121.63
[000.12:56:08.(info)] |online| 30.98          153.6          252.93         272.02         397.31         569.34         2932.74        708697         241244         120.26
[000.12:56:10.(info)] |online| 30.98          155.65         252.93         271.26         393.22         569.34         2883.58        947226         238529         118.97
[000.12:56:12.(info)] |online| 27.01          154.62         252.93         269.85         389.12         548.86         2883.58        1189612        242386         120.95
[000.12:56:14.(info)] |online| 27.01          154.62         252.93         268.98         385.02         532.48         2850.82        1431329        241717         120.74
[000.12:56:16.(info)] |online| 27.01          155.65         252.93         266.98         380.93         512            2342.91        1681684        250355         125.18
[000.12:56:18.(info)] |online| 20.22          155.65         251.9          267.26         380.93         516.1          2375.68        1919753        238069         118.8
[000.12:56:20.(info)] |online| 20.22          155.65         251.9          267.13         380.93         512            2326.53        2160638        240885         120.14
[000.12:56:22.(info)] |online| 20.22          155.65         251.9          267.99         382.98         514.05         2326.53        2392907        232269         115.84
[000.12:56:24.(info)] |online| 20.22          154.62         252.93         269.79         387.07         540.67         2801.66        2614398        221491         110.52
[000.12:56:26.(info)] |online| 20.22          155.65         251.9          269.44         385.02         532.48         2768.9         2855597        241199         120.3
[000.12:56:28.(info)] |online| 20.22          155.65         252.93         270.23         385.02         540.67         2916.35        3084518        228921         114.18
[000.12:56:30.(info)] |online| 20.22          155.65         252.93         270.37         385.02         540.67         2998.27        3319376        234858         117.43
[000.12:56:32.(info)] |online| 20.22          155.65         252.93         270.28         382.98         540.67         3014.66        3557578        238202         118.86
[000.12:56:34.(info)] |online| 20.22          155.65         252.93         269.92         382.98         536.58         3014.66        3799662        242084         120.86
[000.12:56:36.(info)] |online| 20.22          156.67         252.93         269.71         382.98         532.48         2932.74        4039733        240071         119.98
[000.12:56:38.(info)] |online| 20.22          156.67         251.9          269.18         380.93         528.38         2883.58        4285816        246083         122.8
[000.12:56:40.(info)] |online| 20.22          155.65         251.9          268.56         380.93         528.38         2867.2         4534378        248562         123.97
[000.12:56:42.(info)] |online| 20.22          154.62         251.9          268.1          380.93         522.24         2801.66        4781216        246838         123.17
[000.12:56:44.(info)] |online| 20.22          154.62         250.88         267.15         378.88         520.19         2719.74        5038165        256949         128.15
[000.12:56:46.(info)] |online| 20.22          153.6          249.86         266.64         378.88         518.14         2670.59        5287706        249541         124.71
[000.12:56:48.(info)] |online| 20.22          153.6          249.86         266.37         378.88         516.1          2670.59        5533817        246111         122.75
[000.12:56:50.(info)] |online| 20.22          152.57         249.86         268.02         382.98         548.86         2883.58        5738890        205073         102.33
[000.12:56:52.(info)] |online| 20.22          152.57         249.86         268.34         382.98         557.05         2916.35        5970690        231800         115.73
[000.12:56:54.(info)] |online| 20.22          152.57         249.86         268.13         382.98         552.96         2916.35        6214582        243892         121.64
[000.12:56:56.(info)] |online| 20.22          152.57         249.86         268.2          382.98         557.05         2916.35        6451937        237355         118.38
[000.12:56:58.(info)] |online| 20.22          152.57         249.86         268.4          382.98         557.05         2981.89        6685895        233958         116.69
[000.12:57:00.(info)] |online| 20.22          152.57         249.86         268.27         382.98         552.96         2949.12        6927810        241915         120.84
[000.12:57:02.(info)] |online| 20.22          152.57         249.86         268.13         382.98         548.86         2932.74        7170274        242464         121.05
[000.12:57:04.(info)] |online| 40.19          150.53         250.88         269.75         387.07         518.14         2686.98        237420         237420         118.53
[000.12:57:06.(info)] |online| 40.19          152.57         250.88         267.5          380.93         489.47         2039.81        479021         241601         120.5
[000.12:57:08.(info)] |online| 33.53          153.6          251.9          266.4          376.83         479.23         1875.97        717711         238690         119.23
[000.12:57:10.(info)] |online| 33.53          154.62         251.9          264.97         374.78         475.14         1507.33        967649         249938         124.72
[000.12:57:12.(info)] |online| 24.06          155.65         250.88         266.12         376.83         497.66         1818.62        1204216        236567         118.05

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
NumThreads: 32
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
[000.12:57:42.(info)] |online| min (us);      5th (us);      median (us);   avg (us);      95th (us);     99th (us);     99.9th (us);   total_ops;     iter_tops;     tpt (Kops/sec)
[000.12:57:42.(info)] |online| 22.14          148.48         264.19         318.05         651.26         950.27         3571.71        200986         200986         96.44
[000.12:57:44.(info)] |online| 22.14          155.65         258.05         300.76         581.63         856.06         3457.02        435456         234470         117.47
[000.12:57:46.(info)] |online| 22.14          157.7          257.02         295.37         569.34         831.49         3112.96        661100         225644         112.93
[000.12:57:48.(info)] |online| 22.14          159.74         256            294.57         565.25         823.3          3276.8         880693         219593         109.96
[000.12:57:50.(info)] |online| 22.14          159.74         256            293.07         561.15         811.01         3080.19        1103923        223230         111.78
[000.12:57:52.(info)] |online| 22.14          159.74         256            292.12         557.05         806.91         2932.74        1326938        223015         111.68
[000.12:57:54.(info)] |online| 22.14          160.77         256            293.89         561.15         815.1          2932.74        1537012        210074         105.25
[000.12:57:56.(info)] |online| 22.14          160.77         256            292.55         557.05         806.91         2801.66        1763182        226170         113.25
[000.12:57:58.(info)] |online| 22.14          160.77         256            294.15         561.15         819.2          2899.97        1971439        208257         104.39
[000.12:58:00.(info)] |online| 22.14          160.77         256            293.69         561.15         819.2          2998.27        2192712        221273         110.86
[000.12:58:02.(info)] |online| 22.14          160.77         256            292.71         557.05         815.1          2998.27        2419002        226290         113.31
[000.12:58:04.(info)] |online| 22.14          160.77         254.98         291.21         557.05         806.91         2965.5         2651058        232056         116.49
[000.12:58:06.(info)] |online| 22.14          160.77         254.98         289.9          552.96         802.82         2932.74        2884149        233091         116.72
[000.12:58:08.(info)] |online| 22.14          160.77         253.95         288.63         552.96         794.62         2850.82        3118829        234680         117.58
[000.12:58:10.(info)] |online| 22.14          160.77         253.95         287.52         548.86         786.43         2736.13        3353846        235017         117.69
[000.12:58:12.(info)] |online| 22.14          160.77         252.93         286.63         548.86         782.34         2686.98        3587586        233740         117.28
[000.12:58:14.(info)] |online| 22.14          160.77         252.93         286.26         544.77         778.24         2686.98        3816135        228549         114.45
[000.12:58:16.(info)] |online| 22.14          160.77         252.93         285.91         544.77         774.14         2654.21        4044878        228743         114.54
[000.12:58:18.(info)] |online| 22.14          160.77         252.93         285.67         544.77         774.14         2637.82        4272453        227575         114.07
[000.12:58:20.(info)] |online| 22.14          160.77         252.93         286.01         544.77         774.14         2686.98        4491083        218630         109.7
[000.12:58:22.(info)] |online| 22.14          160.77         252.93         285.81         544.77         770.05         2686.98        4718358        227275         113.81
[000.12:58:24.(info)] |online| 22.14          160.77         252.93         287.12         548.86         782.34         2818.05        4919696        201338         101.02
[000.12:58:26.(info)] |online| 22.14          160.77         252.93         288.08         548.86         786.43         2850.82        5125714        206018         103.16
[000.12:58:28.(info)] |online| 22.14          160.77         252.93         288.42         548.86         786.43         2916.35        5341457        215743         108.2
[000.12:58:30.(info)] |online| 22.14          160.77         253.95         288.5          548.86         786.43         2899.97        5562023        220566         110.61
[000.12:58:32.(info)] |online| 22.14          160.77         253.95         288.45         548.86         786.43         2899.97        5785079        223056         111.7
[000.12:58:34.(info)] |online| 22.14          160.77         253.95         288.18         548.86         786.43         2883.58        6012908        227829         114.09
[000.12:58:36.(info)] |online| 22.14          160.77         252.93         287.22         548.86         778.24         2834.43        6256261        243353         121.86
[000.12:58:38.(info)] |online| 22.14          160.77         252.93         287.34         548.86         782.34         2834.43        6476557        220296         110.31
[000.12:58:40.(info)] |online| 22.14          160.77         252.93         287.14         548.86         778.24         2818.05        6702929        226372         113.64
[000.12:58:42.(info)] |online| 23.04          158.72         244.74         261.44         401.41         638.98         905.22         246273         246273         123.32
[000.12:58:44.(info)] |online| 23.04          158.72         243.71         264.08         401.41         634.88         970.75         486520         240247         120.3
[000.12:58:46.(info)] |online| 23.04          159.74         246.78         281.03         528.38         798.72         3719.17        685280         198760         99.53
[000.12:58:48.(info)] |online| 23.04          159.74         247.81         280.8          528.38         786.43         3801.09        914106         228826         114.58
[000.12:58:50.(info)] |online| 23.04          159.74         248.83         280.95         532.48         778.24         2965.5         1141715        227609         113.98


```
