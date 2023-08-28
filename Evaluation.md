# Evaluation

三个全副本，线程池大小 = 16，epoch length = 100 ms，partition_mp = 0，warehouse = 10，limit_txns = unlimit 运行100个epoch的结果 。（16核32G c6）

| 张家口 | Avg Throught / (tps)   |  Avg Latency / (ms)  |
| :--: | :--------------------: | :------------------: |
|A|        18335         |        8581        |
|B|     19162     | 8871 |
|C| 18829 | 8629 |

其中，运行了50个epoch的结果。

|      | Avg Throught / (tps) | Avg Latency / (ms) |
| :--: | :------------------: | :----------------: |
|  A   |        17810         |        4916        |
|  B   |        17189         |        4936        |
|  C   |        17549         |        4726        |

其中，vary 线程池大小 = 32

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        19352         |        7902        |
|   B    |        18047         |        7810        |
|   C    |        19347         |        8315        |

vary limit_txns = 2000  when epoch = 100

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        19523         |        370         |
|   B    |        19470         |        372         |
|   C    |        19408         |        397         |

when epoch = 50

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        18895         |        375         |
|   B    |        18852         |        375         |
|   C    |        18848         |        404         |

vary limit_txns = 3000 epoch = 100

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        20174         |        2850        |
|   B    |        20093         |        2948        |
|   C    |        20135         |        2840        |

epoch = 50

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        19239         |        1634        |
|   B    |        18969         |        1662        |
|   C    |        19284         |        1619        |

仍然有 epoch 堆积现象，经过尝试，epoch长度 = 100ms 时，limit_txns = 2200 时，不会导致epoch堆积

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        21652         |        431         |
|   B    |        21649         |        446         |
|   C    |        21582         |        489         |

vary warehouse = 100 无变化

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |        21646         |        431         |
|   B    |        21628         |        456         |
|   C    |        21615         |        478         |

单副本三分片时，最佳limit_txns = 900 时 不会导致epoch堆积

| 张家口 | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|   A    |         8874         |        430         |
|   B    |         8868         |        445         |
|   C    |         8817         |        506         |





















----

本机与docker

全副本

./server --node_id=1 --thread_num=16 --epoch_length=100 --run_epoch=600 --limit_txns=1200 --warehouse=800

|        | Avg Throught / (tps) | Avg Latency / (ms) |
| :----: | :------------------: | :----------------: |
|  本机  |        10845         |        351         |
| Docker |         9723         |        330         |

分片

