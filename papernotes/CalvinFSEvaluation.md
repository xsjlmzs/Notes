# CalvinFS Evaluation

## Evaluation

```c++
0-----------FillExperiment()
1-----------ConflictingAppendExperiment()
2-----------RandomAppendExperiment()
3-----------CopyExperiment();
4-----------RenameExperiment();
5-----------LatencyExperimentReadFile()
6-----------LatencyExperimentCreateFile()
7-----------LatencyExperimentAppend()
8-----------LatencyExperimentMix()
9-----------LatencyExperimentRenameFile()
10----------CrashExperiment()
```

实验：在不同规模的CalvinFS部署中读取文件、写入文件和创建文件的吞吐量和延迟。

对于每次测量，我们都创建了客户端应用程序，它们发出读取文件、创建文件和写入文件的请求，但频率不同。 对于读取吞吐量的实验，98% 的客户端请求是读取，1% 的操作是文件创建，1% 是写入现有文件。 同样，对于写入基准，客户端提交了 98% 的写入请求，对于附加基准，客户端提交了 98% 的附加请求。

```shell
# 生成pem文件
ssh-keygen -t rsa -b 2048
mv ~/.ssh/id_rsa ~/id_rsa.pem
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 运行实验
./bin/scripts/cluster --command="start" --experiment=5  --clients=100 --max_active=1000 --max_running=100 --local_percentage=100 -ssh_key1="-i ~/id_rsa1.pem" --ssh_key2="-i ~/id_rsa2.pem" --ssh_key3="-i ~/id_rsa3.pem" --calvin_path=/root/CalvinFS

# 查看状态
./bin/scripts/cluster --command="status" --ssh_key1="-i ./id_rsa1.pem" --ssh_key2="-i ./id_rsa2.pem" --ssh_key3="-i ./id_rsa3.pem" --calvin_path=/root/CalvinFS

# kill
./bin/scripts/cluster --command="kill" --ssh_key1="-i ~/id_rsa1.pem" --ssh_key2="-i ~/id_rsa2.pem" --ssh_key3="-i ~/id_rsa3.pem" --calvin_path=/root/CalvinFS
```

```shell
# CalvinDB
# 运行实验
./bin/scripts/cluster --command="start" --lowlatency=0 --type=0 --experiment=0  --percent_mp=0  --percent_mr=0  --hot_records=10000 --max_batch_size=100 --ssh_key1="-i ./id_rsa1.pem" --ssh_key2="-i ./id_rsa2.pem" --ssh_key3="-i ./id_rsa3.pem" --calvin_path=/root/CalvinDB

# kill
./bin/scripts/cluster --command="kill" --lowlatency=0  --type=1 --ssh_key1="-i ./id_rsa1.pem" --ssh_key2="-i ./id_rsa2.pem" --ssh_key3="-i ./id_rsa3.pem" --calvin_path=/root/CalvinDB
```



## Notes

### Abstract

CalvinFS利用分布式数据库系统进行文件系统的元数据管理。有更好的扩展性。

### 1 Introduction

### 2 Background : Calvin

将文件系统的元数据水平分区到多个节点。

CalvinFS的Log(相当于Calvin的Sequencer)实现包括大量“前端”服务器，异步复制的分布式block store，和一小组“metalog” servers。

客户端将请求发送至前端服务器，前端服务器收集一批请求后写入分布式block store。一旦多数block store已复制。前端服务器发送batch id给metalog server。metalog server分布在多个IDC，维护一个基于Paxos复制的"meta-log"，其包括了 a sequence of block ids referencing request batches.

~~存储层需要支持事务语义并知道数据的placement。采用多版本和一致性哈希实现.~~

### 3 CalvinFS Architecture

内存元数据存储。

文件内容存储在非事务性分布式block store中。元数据使用Calvin存储，并将逻辑文件映射到存储其内容的块。

块存储和元数据存储都跨多个数据中心进行复制。

### 4 The CalvinFS Block  Store

### 5 CalvinFS Metadata Management

元数据存储条目KV形式，key为文件的绝对路径，value包括Entry type、Permissions、Contents（对块文件的映射）。

如上所述，CalvinFS 中的元数据管理系统是 Calvin 的一个实例，具有自定义存储层实现，包括复合事务以及原始读/写操作。 它实现了六种事务类型：

- Read(path)
- Create{File,Dir}(path)
- Resize(path, size)
- Write(path, file offset, source, source offset, num bytes)
- Delete(path)
- Edit permissions(path, permissions)

### 6 The Life of an Update

1、客户端发请求给前端，前端插入一个包含数据的data block到CalvinFS的block store。

The front-end begins by performing the write by inserting a data block into CalvinFS’s block store containing the data that will be written to the file. 

第一步时从block store server获取一个unique的block id $\beta$  通过哈希$\beta$ 来确定该block所属的桶。前端找到桶后发送block创建请求。一旦大多数block servers确认了前端的桶创建请求，下一步更新元数据以反应新创建的文件。

接着构建metadata operation：1、create file；2、resize file 3、write $\beta$

将事务请求附加到Calvin Log：1、CalvinFS前端发送请求给一个Calvin Log前端，当Calvin Log前端收集到足够的一批请求后，it is written out another asynchronously replicated block store。等待大多数block servers确认durability后。3、(a)提交batch id附加到Paxos-replicated metalog；(b)按顺序进行处理。4、每个Calvin Shard依照metalog的顺序。



 