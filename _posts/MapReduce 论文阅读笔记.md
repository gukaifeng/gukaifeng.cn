---
title: MapReduce 论文阅读笔记
date: 2021-12-14 14:13:36
updated: 2021-12-19 12:15:00
categories: [论文阅读笔记]
tags: [MapReduce,论文,分布式]
toc: true
mathjax: true
---



MapReduce 论文原文传送门：  
[MapReduce: Simplified Data Processing on Large Clusters](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/16cb30b4b92fd4989b8619a61752a2387c6dd474.pdf)



> 这里需要一个前置知识，你需要先了解一下函数式语言中的 map 和 reduce，因为 MapReduce 就是借鉴了函数式语言中的 map 和 reduce 的思想，如果你不了解这个，后面理解起来可能会有些困难。
>
> 还有就是建议先读 [GFS](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/035fc972c796d33122033a0614bc94cff1527999.pdf) 这篇论文，MapReduce 论文中在一些地方提到了 GFS。不过不是很深入，即便你没读过 GFS 论文，倒也不会有太大的压力。

> 这篇文章不是教学，是原文的翻译，但更是笔记（在很多地方我加了自己的理解，或者以更容易理解的方式改述原文），也许对你有用，也许只有我自己看得懂 =。= ！



## 1. 什么是 MapReduce



MapReduce 是一个编程模型，是一个为了处理与生成大数据集的相关实现。用户指定一个 `map` 函数，处理一个 key/value 键值对，以生成一系列中间 key/value 对，用户再指定一个 `reduce` 函数，合并与所有同一中间 key 的所有中间 value。很多真实世界的任务都可以以这个模型表达出来，论文中就要说这件事。



<!--more-->

以这种函数式风格编写的程序会被自动并行化，在大型计算机集群上执行。run-time 系统负责划分输入数据、规划程序跨一组机器执行、处理机器故障与管理跨机器交流的细节。这样就使得没有任何并行与分布式系统相关经验的程序员可以轻松的使用大型分布式系统资源。

MapReduce 的实现运行在一个大型计算机器群上，并且是高可伸缩的：一个典型的 MapReduce 计算是在数千台机器上处理数 TB 的数据。程序员会发现这个系统非常易用：已经有数百个实现好的 MapReduce 程序，并且每天有超过一千个 MapReduce 任务在 Google 的集群上执行。（这数据是 04 年论文上的，现在多的多了。）





## 2. 编程模型



计算程序拿到一组输入 key/value 对，然后产出一组输出 key/value 对。MapReduce 库的用户可以把这个计算表达为两个函数：*Map* 和 *Reduce*。

*Map*，由用户编写，获取一个输入 kv 对，然后生成一组中间 kv 对。MapReduce 库把同一中间 key `I` 的所有中间 value 组合到一起，发送给 *Reduce* 函数。

*Reduce* 函数也由用户编写，接收一个中间 key `I` 和这个 key 的一组 value。*Reduce* 合并这些 value，组合成一组可能更小的 value（每次 *Reduce* 调用往往只产出 0 个或者 1 个输出 value）。这些中间 value 通过一个迭代器传递给用户的 reduce 函数，这样使得我们可以处理那些总量很大，无法全部放入内存的 value。



### 2.1. 示例

假设我们要实现一个程序，这个程序要在大量的文档中计算每个单词出现的次数。

用户编写的代码可能类似下面的伪代码：

```cpp
map(String key, String value):
  // key: document name
  // value: document contents
  for each word w in value:
    EmitIntermediate(w, "1");
reduce(String key, Iterator values):
  // key: a word
  // values: a list of counts
  int result = 0;
  for each v in values:
    result += ParseInt(v);
  Emit(AsString(result));
```

*map* 函数给每个单词相关的出现计数加 1。*reduce* 函数给一个特定单词的所有计数求和。

用户编写代码，使用输入和输出文件的名字填充一个 *mapreduce specification（mapreduce 特定的一个规范形式）* 对象，并可选的调整参数。然后用户调用 *MapReduce* 函数，给其传递刚刚的 *specification* 对象，用户的代码就被链接到 MapReduce 库中（C++ 实现）。



\-

这个示例完整的代码在论文最后的 Appendix A，这里直接在下面给出（这个示例第 66 行还用到了 4.3 要讲的组合器函数）。

这个程序用于在一组输入文件中统计每个单词出现的次数，输入文件通过命令行给出。

```cpp
#include "mapreduce/mapreduce.h"
// User’s map function
class WordCounter : public Mapper
{
public:
    virtual void Map(const MapInput &input)
    {
        const string &text = input.value();
        const int n = text.size();
        for (int i = 0; i < n;)
        {
            // Skip past leading whitespace
            while ((i < n) && isspace(text[i]))
                i++;
            // Find word end
            int start = i;
            while ((i < n) && !isspace(text[i]))
                i++;
            if (start < i)
                Emit(text.substr(start, i - start), "1");
        }
    }
};
REGISTER_MAPPER(WordCounter);
// User’s reduce function
class Adder : public Reducer
{
    virtual void Reduce(ReduceInput *input)
    {
        // Iterate over all entries with the
        // same key and add the values
        int64 value = 0;
        while (!input->done())
        {
            value += StringToInt(input->value());
            input->NextValue();
        }
        // Emit sum for input->key()
        Emit(IntToString(value));
    }
};
REGISTER_REDUCER(Adder);
int main(int argc, char **argv)
{
    ParseCommandLineFlags(argc, argv);
    MapReduceSpecification spec;
    // Store list of input files into "spec"
    for (int i = 1; i < argc; i++)
    {
        MapReduceInput *input = spec.add_input();
        input->set_format("text");
        input->set_filepattern(argv[i]);
        input->set_mapper_class("WordCounter");
    }
    // Specify the output files:
    // /gfs/test/freq-00000-of-00100
    // /gfs/test/freq-00001-of-00100
    // ...
    MapReduceOutput *out = spec.output();
    out->set_filebase("/gfs/test/freq");
    out->set_num_tasks(100);
    out->set_format("text");
    out->set_reducer_class("Adder");
    // Optional: do partial sums within map
    // tasks to save network bandwidth
    out->set_combiner_class("Adder");
    // Tuning parameters: use at most 2000
    // machines and 100 MB of memory per task
    spec.set_machines(2000);
    spec.set_map_megabytes(100);
    spec.set_reduce_megabytes(100);
    // Now run it
    MapReduceResult result;
    if (!MapReduce(spec, &result))
        abort();
    // Done: ’result’ structure contains info
    // about counters, time taken, number of
    // machines used, etc.
    return 0;
}
```





### 2.2. 类型

尽管上面的伪代码（这里先不看那个具体实现代码）是用 string 类型的输入和输出编写的，但在概念上，用户提供的 map 和 reduce 函数有着相关的类型：

```
map      (k1, v1)         -> list(k2, v2)
reduce   (k2, list(v2))   -> list(v2)
```

也就是说，输入的 key 和 value 与 输出的 key 和 value 来自不同的域。再进一步，中间的 key 和 value 与输出的 key 和 value 来自相同的域。

上面的 C++ 实现中，不论是给用户定义的函数传递的，还是从用户定义的函数中接收的，都是 string 类型。将 string 类型与实际合适的类型的转换工作留给了用户来完成。





### 2.3. 更多例子

论文中给出了几个有趣的程序，这几个程序也可以很简单的使用 MapReduce 计算表达出来。



**Distributed Grep:** 分布式 Grep。Grep 是一个从文本中查找包含指定模式（模式匹配，例如字符串）的行的工具。map 函数 emit 一行，如果这行包含指定的模式，这些行就是中间数据。reduce 函数负责拷贝这些行，并把它们输出。



**Count of URL Access Frequency:** URL 访问计数。map 函数处理 web 的 request 的 log，然后输出 <URL, 1>（即给对应的计数加 1，这个与之前的统计文章中单词出现次数类似）。reduce 函数把同一个 URL 所有的统计好的值加起来，然后 emit 一个 <URL, 总访问计数> 对。





**Reverse Web-Link Graph:**





**Term-Vector per Host:**





**Inverted Index:** 倒排索引。







**Distributed Sort:** 分布式排序。







## 3. 实现





MapReduce 接口可能有非常多种不同的实现，选择正确的实现依赖于具体的环境。例如，一种实现可能适用于小型共享内存机器，另一种适用于大型 NUMA 多处理器，而另一种适用于更大的联网机器集合。





这里要描述的是一种针对 Google 广泛使用的计算环境的实现：**使用交换式以太网连接到一起的大型商用 PC 集群。**

这个环境有以下特征：

1. 机器一般是双处理器的（两个 x86 处理器），运行 Linux 系统。每个机器 2-4 GB 内存。
2. 使用商业网络硬件，在机器层面，一般有 100Mb/s 或 1Gb/s，但在整体二分带宽中平均要少得多。
3. 一个集群包含数百数千个机器，因此有某个或某些机器故障是很常见的。
4. 由直接连接到单个机器上的廉价的 IDE 磁盘提供存储。一个内部开发的分布式文件系统用于管理这些存在磁盘上的数据，这个分布式文件系统使用复制以在不可靠的硬件上保证可用性和可靠性。
5. 用户提交作业给调度系统。每个作业包含一组任务，调度程序将作业映射到集群中一组可用的机器上。







### 3.1. 执行过程总览



*Map* 调用通过自动地划分输入数据为一组 $M$ 个 *split* 分布在多个机器上，这些输入 split 可以在不同的机器上并行处理。*Reduce* 调用分布式的，其通过使用划分函数（例如 *hash(key) mod R*）划分中间 key 空间为 R 个 piece 来实现，R 的值和划分函数都是由用户指定的。

Figure 1 展示了在我们的实现中，一个 MapReduce 操作的总体流程。

![Figure 1: Execution overview](https://gukaifeng.cn/posts/mapreduce-lun-wen-yue-du-bi-ji/MapReduce_Figure_1.png)

当我们的程序调用 *MapReduce* 函数时，会依次发生下面的事（图中的编号标签与下面的序号相对应）：

1. 用户程序中的 MapReduce 库先把输入文件划分为 $M$ 个 split，每个 split 的大小一般是 16-64 MB（用户可以通过一个可选的参数控制）。然后 MapReduce 在一个机器集群上启动很多个程序的副本。
2. 这些程序的副本中有一个是特殊的，叫做 master。master 给其他的 workers 分配工作。一共有 $M$ 个 map 任务和 $R$ 和 reduce 任务要分配，master 挑选出闲置的 workers，给每个闲置的 worker 指派一个 map 任务或 reduce 任务。
3. 一个被指派了 map 任务的 worker 读对应的输入 split 的内容。其从输入数据中解析出 key/value 对，然后将每个 key/value 对传递给用户定义的 *Map* 函数，由 *Map* 函数生成的中间 key/value 对被缓存在内存中。
4. 内存中缓存的中间 key/value 对会被周期性的写入本地磁盘，并且会被划分函数划分为 $R$ 个部分。然后这些中间 key/value 在磁盘上的位置会被传递回 master，master 再把这些位置信息传递给 reduce workers。
5. 当一个 reduce worker 收到了 master 提供的这些中间 key/value 对在磁盘上的位置信息，其就会使用远程过程调用(RPC)，从 map workers 的本地磁盘中读取缓存的中间 key/value 对数据。当一个 reduce worker 读完了全部的中间数据，就会对这些中间数据按 key 排序，这样同一个 key 的所有数据就都放在了一起。排序是很有必要的，因为会有很多不同的 keys 映射到同一个 reduce 任务中。如果中间数据太大，无法全部放入内存，就会使用外部排序。
6. reduce worker 遍历排序好的中间数据，将遇到的每个不同的中间 key 与与之对应的中间 values 集合传递给用户定义的 *Reduce* 函数。*Reduce* 函数的输出被附加到一个这个 reduce 分块的最终输出文件中。
7. 当全部的 map 任务和 reduce 任务全部完成后，master 会唤起用户程序。此时，用户程序中的 *MapReduce* 调用结束，将继续执行用户程序中后面的代码。



当所有的操作全部成功完成时，mapreduce 执行的输出是可用的，且被划分进 $R$ 个输出文件（每有一个 reduce 任务，就会有一个输出文件，文件名由用户指定）。

一般来说，用户不需要组合这 $R$ 个输出文件为一个。用户常常会把这些输出文件作为输入传递给另一个 MapReduce 调用，或者在另一个分布式应用（可以处理划分进多个文件的输入的分布式应用）中使用这些输出文件。





### 3.2. Master 数据结构



master 中维护了几个数据结构。对于每个 map 任务和 reduce 任务，master 中存储了其状态（*空闲*、*处理中*，或*已完成*），还有 worker 机器（非空闲的）的标识。

master 是从 map 任务向 reduce 任务传递中间文件区域位置的管道，因此，对于每个已完成的 map 任务，master 要存储由这个 map 任务生成的 $R$ 个中间文件区域的位置和大小。当 map 任务完成时，master 接收到中间文件区域的位置和大小更新信息，这些信息会逐步推送给正在处理 reduce 任务的 worker。





### 3.3. 容错



因为 MapReduce 库是被设计用来在数百或数千台机器上帮助处理非常大量的数据的，所以这个库必须能优雅地容忍机器的故障。



**Worker 故障**

Master 会定期 ping 每个 worker，如果在一定时间内没有收到某个 worker 的应答，master 就会把这个 worker 标记为故障。任何由这个故障的 worker 完成的 map 任务都会被重设为初始的*空闲*状态（未开始处理的状态），这样就可以由其他 worker 来重新调度。同样的，当一个 worker 发生故障时，任何正在由此 worker 处理的 map 和 reduce 任务都会重设回初始*空闲*状态，然后可以被其他 worker 重新调度。



由故障的 worker 已经完成的 map 任务之所以也要重新执行，是因为这些 map 任务的输出是存在故障机器的本地磁盘上的，机器已经故障了，本地磁盘的里的东西也拿不出来了。而已由故障机器完成的 reduce 的任务就不需要重新执行，因为这些 reduce 任务的输出是存在全局文件系统(GFS)中的。（关于 GFS，有论文 [The Google File System](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/035fc972c796d33122033a0614bc94cff1527999.pdf)，或许你也可以看看我的 [GFS 论文阅读笔记](https://gukaifeng.cn/posts/gfs-lun-wen-yue-du-bi-ji/) 或其他资料。）

当一个 map 任务被 worker A 执行了，然后又被 worker B 执行了（由于 A 故障了）的时候，所有正在 workers 上执行的 reduce 任务都会重新执行。如果一个 reduce 任务还没开始从 A 读数据，那就直接从 B 读。



MapReduce 对大规模 worker 故障具有弹性（即可以快速恢复）。例如，在 MapReduce 操作期间，正在运行的集群由于网络维护导致了一次 80 个机器在几分钟内同时变得不可达。MapReduce master 简单地重新执行了无法访问的工作机器所做的工作，并继续向前推进，最终完成 MapReduce 操作。



**Master 故障**

让 master 定期写入上述 master 数据结构的检查点是很容易的。如果 master 任务死掉了，就会有一个新的 master 副本从最近的检查点状态启动。然而，考虑到我们只有一个 master，所以它不太可能故障。如果 master 故障了，我们的实现会终止 MapReduce 计算，客户端可以检查这个情况，然后根据需要重试 MapReduce 操作。



**故障出现时的语义**

当用户提供的 *map* 和 *reduce* 运算符是其输入值的确定性函数时，我们的分布式实现产生的输出，与没有发生故障的、按顺序执行的整个程序的输出是一样的。

我们依赖 map 和 reduce 任务输出的原子提交来实现上述这点。每个正在处理的任务将其的输出写入一个私有的临时文件。一个 reduce 任务产生一个这样的文件。一个 map 任务产生 $R$ 个这样的文件（每有一个 reduce 任务，就有一个这样的文件）。当一个 map 任务完成时，其 worker 会发送一条信息给 master，信息中心包含了 $R$ 个临时文件的名字。如果 master 收到了一个已经完成的 map 任务的完成信息，master 就会忽略这个信息；否则，master就会在 master 的数据结构中记录下这 $R$ 个文件的名字。

当一个 reduce 任务完成时，其 worker 会原子地重命名其临时输出文件为最终输出文件。如果同一个 reduce 任务被执行在多个机器上，那么将为同一个最终输出文件执行多个重命名调用。我们依赖由底层文件系统提供的原子重命名操作，以确保最终的文件系统状态只包含由一个 reduce 任务执行产生的数据。

我们的 *map* 和 *reduce* 操作符中的绝大多数都是确定的，在这种情况下，我们的语义与按顺序执行程序是一样的（即分布式地执行程序各个部分与直接按顺序执行程序的每一个部分，结果一定是一样的），这就使得我们的程序员可以很容易地推断他们的程序的行为。

当 *map* 和/或 *reduce* 操作符不是确定的，我们会提供较弱但仍然合理的语义。在操作符非确定的这种情况下，可能特定 reduce 任务 $R_1$ 的输出与这个非确定程序按一种顺序执行产生的 $R_1$ 的输出相同，而 $R_2$ 的输出可能与这个非确定程序按另一种不同的顺序执行产生的 $R_2$ 的输出相同。

举一个例子，有一个 map 任务 $M$ 和两个 reduce 任务 $R_1$ 和 $R_2$，设 $e(R_i)$ 是已经提交了的 $R_i$ 的执行（一个确定的执行）。这是就会产生较弱的语义，因为 $e(R_1)$ 可能读由 $M$ 的一种执行产生的输出，而 $e(R_2)$ 可能读 $M$ 的另一种执行产生的输出。





### 3.4. 输入数据的位置

在我们的计算环境中，网络带宽是一种相对短缺的资源。

我们通过优化存储在组成我们的集群的机器的本地磁盘上的输入数据来节省网络带宽（通过 GFS 管理。关于 GFS，有论文 [The Google File System](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/035fc972c796d33122033a0614bc94cff1527999.pdf)，或许你也可以看看我的 [GFS 论文阅读笔记](https://gukaifeng.cn/posts/gfs-lun-wen-yue-du-bi-ji/) 或其他资料）。

GFS 将每个文件划分为多个 64MB 的块，每个块都会存几个副本（一般是 3 个）在不同的机器上。

MapReduce 的 master 会记录这些输入文件的位置信息，然后尝试将一个 map 任务调度给一个有相应输入数据副本的机器。如果失败了（比如这些机器都不是闲置的），master 就尝试把这个 map 任务调度给一个离有相应输入数据副本的机器比较近的机器（例如，在与包含数据的机器位于同一网络交换机上的工作机器上）。

当在一个集群中的大部分 worker 上运行大型的 MapReduce 操作时，大部分输入数据都是从本地读的，也就是不消耗网络带宽。



### 3.5. 任务粒度



> 在并行计算中，任务的粒度指的是对该任务执行的工作量的度量。

我们之前描述的，把 map 阶段和 reduce 分别细分为了 $M$ 和 $R$ 个部分，理想情况下，$M$ 和 $R$ 的值应该是远远大于 worker 机器总数的。让每个 worker 执行很多不同的任务可以改善动态平衡，也可以加快当一个 worker 发生故障时的恢复速度：由故障 worker 完成的很多 map 任务可以分布到所有的其他 worker 机器上（我的理解是故障 worker 完成的很多 map 任务的输出结果肯定是拿不到了，这些 map 任务都要重新执行，但是可以把这些 map 分散到各个机器一起执行，这样就会比较快）。

在我们的实现中，$M$ 和 $R$ 的大小有一些实际的限制，因为 master 必须做出 $O(M+R)$ 的调度决策，并且像上面所说的那样，在内存中保持 $O(M*R)$ 状态。（$O(M\*R)$ 看着挺吓人的，但实际上内存使用量很小，状态的 $O(M\∗R)$ 块大约由每个 map/reduce 任务对的一个字节的数据组成。）

此外，$R$ 常常由用户限制，因为每个 reduce 任务的输出都被分隔到了不同的输出文件。事实上，我们一般让单个 map 任务大概使用输入数据中 16-64 MB，也就是 $M$ 的值是输入数据的总大小除以每个任务使用的输入数据大小（这样会使得我们在 3.4 中说的位置优化就会非常高效）。$R$ 的值一般设定为我们期望使用的 worker 机器数量的较小倍数。例如，执行一个 MapReduce，$M=200,000$，$R=5,000$，worker 机器数量 2000 就比较合理。





### 3.6. 候补任务



导致 MapReduce 操作耗费的时间变长的常见原因之一是有一个“掉队者(straggler)”：一个机器花费了比平时更长的时间完成了计算中的最后几个 map 或 reduce 任务中的一个。掉队者出现的原因有很多。例如，一个机器的磁盘是坏的，频繁的出现一些可校正的错误（意思就是这个错误可以恢复），使得这个磁盘的读性能从 30 MB/s 降到了 1 MB/s。而集群调度系统可能把一些其他任务也分给这个台机器，就导致这些任务互相竞争 CPU、内存、本地磁盘或者是网络带宽，这样 MapReduce 代码的执行速度就更慢了。一个我们最近遇到的问题是一个机器初始化代码中的 bug，这个 bug 导致处理器 cache 变得不可用：受影响的机器的计算减慢了一百多倍。

我们有一个通用的机制来减轻掉队者的问题。当一个 MapReduce 操作快要完成时，master 调度剩下的正在处理中的任务的候补执行（就是再执行一份同样的任务）。主任务（原来的任务）和候补任务只要有一个完成了，这个任务就会被标记为已完成。我们调整过这个机制，使其执行候补任务导致的计算资源增加不超过几个百分点。我们发现这种机制能大大减少大型 MapReduce 操作的耗时。例如，5.3 节中的排序程序，不使用候补任务机制比使用了候补任务机制的耗时长了 44%。





## 4. 改进



尽管在大多数需求下，简单的 map 和 reduce 函数提供的基础的功能性已经足够了，但是我们还是发现了几个有用的扩展。这一节将讲述这些扩展内容。



### 4.1. 划分函数



MapReduce 的用户指定 reduce 任务和输出文件数 *R*。数据会依据中间 key，被划分函数划分给这些任务（*R* 个任务，也就是数据会被划分成 *R* 份）。一个默认的划分函数就是使用哈希（例如 *hash(key) mod R*）。这样的划分结果是相当均匀的，但是有时候使用一切其他的划分方法来划分也是非常有用的。例如，我们的输出 key 都是 URL，我们想要所有属于同一个 host 的条目最终都在同一个输出文件中，我们就可以使用一个特殊的划分函数，比如像 *"hash(Hostname(urlkey)) mod R"*，Hostname() 函数将 URL 中的 host 提取出来，使用 host 进行哈希，就可以使所有属于同一个 host 的 URL 最终输出到同一个输出文件了。





### 4.2. 顺序保证

我要确保在一个给定的划分内部，中间 key/value 对都已经被处理为 key 递增有序的。保证了这个顺序，会使得确保每个划分的输出都是有序的变得简单。这是非常有用的，例如输出文件的格式需要支持 key 的高效的随机查找，或者用户发现如果输出是有序的会很方便（做后续的操作）。







### 4.3. 组合器(conbiner)函数



在某些情况下，每个 map 任务产生的中间键中有大量的重复，用户指定的 *Reduce* 函数是可交换的、关联的。

一个很好的例子就是 2.1 中的单词统计示例。由于单词重现的频率往往服从 Zipf 分布，所以每个 map 任务会产生成百上千的格式化记录 *<the, 1>*。所有的这些计数都会通过网络发送给同一个 reduce 任务，然后由 *reduce* 函数将这些计数加起来，得出最终的一个总数值。

我们可以让用户指定一个组合器(*combiner*)函数，在这些数据发送到网络之前进行部分合并。

组合器函数执行在每个运行了 map 任务的机器上，通常组合器函数的代码和 reduce 函数的代码是一样的（因为都是对 map 的结果做同样的操作）。组合器函数和 reduce 函数唯一的不同就是 MapReduce 如何操作这两个函数的输出。reduce 函数的输出会写入最终的输出文件中，而组合器函数的输出会写入中间文件中，随后会被发送给一个 reduce 任务。



部分组合大大加速了某些类别的 MapReduce 操作的速度。论文最后的 Appendix A 代码中就使用了组合器函数。





### 4.4. 输入和输出类型



MapReduce 库对于输入数据，支持几种不同的格式。例如 “text” 格式，会把输入的每一行看作一个 key/value 对：key 是文件中的偏移量，value 是行的内容。另一个支持的常见格式中，存储的是 key/value 对的序列，这些 key/value 对是根据 key 排序了的。每种输入类型的实现都清楚如何划分输入数据为有意义的部分，使得这些部分可以由 map 任务来处理（例如 "text" 格式的输入数据，只在一行的边界处划分）。用户可以通过提供一个实现了 *reader* 接口的实现来添加一个新的输入类型支持，不过大部分用户只会用到少数预定义的输入类型中的一种。

reader 不一定非得是用文件中读取数据，也可以是数据库，或者是映射到内存中的数据结构中。

与输入类型类似，我们也支持一些输出类型，用于以不同的格式生成数据。同样的用户也可以添加新的输出类型支持。





### 4.5. 副作用(side-effects)

在某些情况下，MapReduce 的用户发现从他们的 map 和/或 reduce 运算符中生成辅助文件作为附加输出很方便。我们依靠应用程序编写器(application writer)来使这种副作用原子化和幂等。 通常，应用程序写入一个临时文件，并在该文件完全生成后自动重命名该文件。

我们不支持由单个任务生成的多个输出文件的原子两阶段提交。 因此，生成具有跨文件一致性要求的多个输出文件的任务应该是确定性的。这种限制在实践中从未成为问题。

> 在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。幂等函数，或幂等方法，是指可以使用相同参数重复执行，并能获得相同结果的函数。 这些函数不会影响系统状态，也不用担心重复执行会对系统造成改变。





### 4.6. 跳过错误记录

有些时候由于用户代码中的 bug 导致 *Map* 或 *Reduce* 函数在遇到某些记录时确定性崩溃。这样的 bug 使得我们的 MapReduce 操作无法完成。通常的做法是修复这个 bug，但有些时候这是不可行的（比如 bug 存在于第三方库中，我们又无法修改这个库的源码）。有些时候，忽略几条记录也是可以接受的，例如我们要读取一个非常大的数据集，然后做统计分析，少几条记录几乎不会影响最终结果。我们提供一个可选的执行模式，使得 MapReduce 库检测到会导致确定性崩溃的记录时跳过这些记录，使得进程得以继续推进。

每个 worker 进程都设置一个信号处理器(signal handler)，这个处理器可以捕获段违规(segmentation violation)和总线错误。在调用一个用户的 *Map* 或 *Reduce* 操作前，MapReduce 库会将参数的序列号存在一个全局变量中。如果用户代码产生了一个信号，信号处理器就会发送一个 "last gasp"（意为最后一口气儿） UDP 包给 master，包中包含了这个序列号。当 master 发现某个特定的记录有超过 1 个故障时，就会明白，当后面重新执行相应的 map 或 reduce 任务时，应该跳过这个记录。



### 4.7. 本地执行

调试 Map 或 Reduce 函数是很棘手的，因为实际的计算是发生在分布式系统中的，通常有几千台机器，并且具体任务的指派也是由 master 动态决定的。为了使调试、分析、小规模测试更容易，我们还开发了 MapReduce 库的一个替代实现，这个实现会在一台本地机器上顺序执行所有的 MapReduce工作。并且给用户提供了控件，用户可以把计算限制在一个特定的 map 任务中。用户使用一个特殊的 flag 调用他们的程序，然后就可以轻松地使用他们认为有用的调试或者测试工具了（例如 GDB）。





### 4.8. 状态信息

master 中运行了一个内部 HTTP 服务器，并且会导出一些列的状态页面，这些状态页面是用来给人看的。

状态页面显示了计算进程信息，比如多少个任务已经完成、多少个任务正在处理中、输入的字节数、中间数据的字节数、输出的字节数、处理的速率等等。这个页面也包含了每个任务生成的标准输出和标准错误输出文件的链接。

用户可以用这些数据来预测计算要花费的时间、是否需要给计算提供更多的资源等等。这些状态页面也可以用来判断计算是否比预期的要慢很多。

顶级状态页面额外显示了那些 worker 失败了，失败时正在处理哪个 map 或 reduce 任务。这个信息对于尝试诊断用户代码中的 bug 时很有用。





### 4.9. 计数器

MapReduce 库提供了一个计数器工具，用于统计各种事件发生的次数。例如用户代码可能想统计处理过的单词的数量，或者索引过的德语文档的数量，等等。

为了使用我们提供的计数器工具，用户代码中需要创建一个名为 "counter" 的对象，然后在 *Map* 和/或 *Reduce* 函数中合适的位置更新计数。例如：

```cpp
Counter* uppercase;
uppercase = GetCounter("uppercase");

map(String name, String contents):
  for each word w in contents:
    if (IsCapitalized(w)):
      uppercase->Increment();
    EmitIntermediate(w, "1");
```

这段伪代码中，在 map 任务中统计了所有大写单词的数量。

单个 worker 机器会定期将计数器的值传给 master（附加在 ping 消息的应答消息中）。当 MapReduce 操作完成时，master 累加来自成功完成的 map 和 reduce 任务的值，将累加结果返回给用户代码。

当前的计数器值也可以在 master 状态页看到，人们可以看到实时的计算过程。

master 累加计数器值的时候，会排除同一个 map 或 reduce 任务重复执行（例如候补任务，或者由于故障导致重新执行的任务）的影响以避免重复计数。

一些计数器的值是由 MapReduce 库维护的，比如处理的输入 key/value 对数量，输出 key/value 对的数量。

用户会发现计数器工具对检查 MapReduce 操作的行为非常有用。例如在某些 MapReduce 操作中，用户代码想要确保输出的 key/value 对数量与输入的数量完全相等，或者确保处理德语文档与处理全部文档的数量的比例在一个可以接受的范围内。







## 5. 性能

这一部分我们通过两个运行在大型计算机集群上的任务来测试 MapReduce 的性能。

一个是模式匹配 Grep，在大约 1TB 的数据中查找一个指定的模式。另一个是 Sort，排序大约 1TB 的数据。



在由 MapReduce 的用户编写的真实程序中，这两个程序是大数据集的代表。Sort 代表一类 —— 将数据重排，数据从一种形态变为另一种形态。Grep 代表一类 —— 从一个大数据集提取一小部分我们感兴趣的数据。



### 5.1.  集群配置

所有的程序都执行在包含大约 1800 台机器的一个集群上。每个机器配备两个 2GHz Inter Xeon 处理器（启用超线程(Hyper-Thread)技术）、4 GB 内存、160 GB 的 IDE 磁盘，以及千兆以太网链路。这些机器被安排在一个两级树形交换网络中，在 root 处有大约 100-200 Gbps 的总带宽可用。 所有机器都在同一个 host 设备中，因此任何机器两两之间的往返时间都小于一毫秒。

### 5.2. Grep

*grep* 程序扫描 $10^{10}$ 个 100 字节大小的记录，找出一个出现次数相当少的，有 3 个字符的字符串（在 92337 个记录中出现过，92337 次看着挺多，但是相比于总共的 $10^{10}$ 条记录，就相当少了）。输入被划分为大约为 64 MB 大小的块（即 $M = 15000$），所有的输出都被放进一个文件中（$R=1$）。

Figure 2 显示了随着时间的推移，grep 程序计算的过程。

![Figure 2: Data transfer rate over time](https://gukaifeng.cn/posts/mapreduce-lun-wen-yue-du-bi-ji/MapReduce_Figure_2.png)

Y 轴是扫描输入文件的速率。这个速率随着越来越多的机器被指派给 MapReduce 计算逐渐增加，当 1764 个 worker 被指派参与计算时，速度达到峰值，超过 30 GB/s。随着 map 任务的完成，扫描输入文件的速率开始下降，在计算进行到大约 80 秒是，速率跌至 0。

这个程序的整个计算过程，从开始到结束大约花了 150 秒。这 150 秒中还包括了大约 1 分钟的启动开销，这是由于要将程序传播到所有 worker 机器上，以及延迟与 GFS 交互以打开 1000 个输入文件的集合并获得文件位置优化所需的信息。



### 5.3. Sort

*sort* 程序排序 $10^{10}$ 个 100 字节的记录（总共约 1TB 的数据）。这个程序是模仿 TeraSort 基准测试的。

这个排序程序包含不超过 50 行的用户代码。一个三行的 *Map* 函数从文本行中提取一个 10 字节的排序 key，然后 emit 这个排序 key 和原始的文本行作为中间 key/value 对。我们使用内置的 *Identity* 函数作为 *Reduce* 运算符。
此函数传递未修改过的中间键 key/value 对作为输出 key/value 对。 最终排序的输出被写入一组 2 路复制的 GFS 文件（即，有 2 TB 被写入作为程序的输出）。

在计算开始之前，我们把输出数据划分为 64MB 大小的块（$M=1500$），把排序后的输出划分进 4000 个文件中（$R=4000$）。划分函数使用 key 开头的一些字节来把其划分进 R 个块中的一个。

我们这个基准测试的划分函数是已经知道 key 的分布的。在一个通用的排序程序中，我们会添加一个预传递 MapReduce 操作，这个预传递操作会收集 key 的样本然后使用样本中 key 的分布信息来计算最终排序传递的划分点。

![Figure 3: Data transfer rates over time for different executions of the sore program](https://gukaifeng.cn/posts/mapreduce-lun-wen-yue-du-bi-ji/MapReduce_Figure_3.png)

Figure 3 (a) 展示了一个排序程序的正常执行的过程。

左上角的图片显示了读取输入的速率，峰值达到了大约 13 GB/s。由于 map 任务全部完成，这个速率在时间进行到 200 秒前极速下降直至 0。注意一下这个 *sort* 程序的读取输入的速率要小于 *grep* 程序的，这是因为 *sort* 程序的 map 任务要花一半的时间和 I/O 带宽用于将中间数据写入本地磁盘，而 *grep* 程序对应的中间输出非常非常小。

左边中间的图片显示了从 map 任务经过网络向 reduce 任务发送数据的速率。数据的重排在第一个 map 任务完成时开始。图中的第一个驼峰是由第一批大约 1700 个 reduce 任务导致的（整个 MapReduce 被指派给了大约 1700 台机器，每个机器在同一时刻最多执行一个 reduce 任务）。在大约 300 秒的计算后，一部分第一批的 reduce 任务已经完成，我们开始为剩下的 reduce 任务重排数据。全部的重排大约在 600 秒的计算内完成。

左下角的图显示了 reduce 任务将排序好的数据写入最终输出文件的速率。在第一轮重排结束与一轮写的开始有一段间隔（看 (a) 图二第一个驼峰结束，和 (a) 图三的第一个驼峰开始），这是因为这些机器正忙于排序中间数据。写入操作以大约 2-4 GB/s 的速度持续了一段时间。所有的写在在计算进行到大约 850 秒的时候全部完成。包含启动开销，整个计算共耗时 891 秒，这个与目前最好的由 TeraSort benchmark 报告的结果 1057 秒接近。

要注意几个事情：输入速率高于重排速率和输出速率是因为我们的本地优化 —— 大部分数据是从一个本地磁盘读取的，绕开了我们带宽相对受限的网络。重排速率高于输出速率是因为输出阶段要写排好序的数据的两份副本（出于可靠性和可用性的原因，我们写两份副本是因为我们底层文件系统提供的可靠性与可用性机制）。如果底层文件系统使用抹除码(Erasure code)而不是复制，写数据需要的网络带宽要求就变低了。



### 5.4. 候补任务的影响

Figure 3 (b) 展示了一个禁用了候补任务的排序程序执行的过程。


这个执行流程和 Figure 3 (a) 中的类似，除了 (b) 中的流程有一个非常非常长的尾巴（几乎没有任何写操作发生），在 960 秒后，除了 5 个 reduce 任务，其他的 reduce 任务全部都完成了。然而这几个掉队者直到 300 秒后才完成，整个计算过程耗时 1283 秒，相比启用了候补任务的执行增加了 44% 的时间。





### 5.5. 机器故障的影响

在 Figure 3 (c) 中，我们在计算开始几分钟后，故意杀死了 1746 个 worker 进程中的 200 个。底层的集群调度器立即在这些机器上重启新的 worker 进程（我们只是杀死了 worker 进程，机器还是正常运转的）。

我们以负数输入速率显示 worker 进程死亡，这是由于一些先前由 map 任务完成的工作消失了（因为相应的 map 任务被杀死了），需要重新做。map 工作的重新执行开始的是相对较快的。整个计算在 933 秒的时候完成（包含了启动开销）。在机器发生故障的情况下，耗时仅比正常执行增加了 5%。





## 6. 经验

我们在 2003 年 2 月编写了 MapReduce 库的第一个版本，并在 2003 年 8 月进行了巨大的升级，包括本地优化、跨 worker 机器任务执行的动态负载均衡等等。从那时起，我们非常惊讶，这个 MapReduce 库在我们工作中的各种问题上是如此的广泛的可用。在 Google 内部，MapReduce 已经在很多个领域广泛使用了，包括：

* 大规模机器学习问题，
* Google News 和 Froogle 产品的聚类问题，
* 提取用于生成流行查询报告的数据（例如 Google Zeitgeist），
* 为新的实验和产品提取网页的属性（例如，从大型网页语料库中提取地理位置以进行本地化搜索），以及
* 大规模图计算。



![Figure 3: MapReduce instances over time](https://gukaifeng.cn/posts/mapreduce-lun-wen-yue-du-bi-ji/MapReduce_Figure_4.png)

Figure 4 显示了从我们主要源码管理系统的中得到的，随着时间的推移，独立的 MapReduce 程序数量的巨大增长，从 2003 年早期的 0 到 2004 年 9 月下旬的大概 900 个独立的实例。MapReduce 之所以能够如此成功，是因为 MapReduce 使得在半小时内编写一个简单的程序并在一千台机器上高效运行成为了可能，大大缩短了开发和成型的周期。此外，MapReduce 也使得没有任何分布式和/或并行系统经验的程序员可以轻松地将大量的资源利用起来。





![Table 1: MapReduce jobs run in August 2004](https://gukaifeng.cn/posts/mapreduce-lun-wen-yue-du-bi-ji/MapReduce_Table_1.png)



在每个作业结束时，MapReduce 库会记录有关作业使用的计算资源的统计信息。 在 Table 1 中，我们显示了 2004 年 8 月在 Google 运行的 MapReduce 作业子集的一些统计数据。





### 6.1. 大规模索引

至今（论文发布时间是 2004 年），我们关于 MapReduce 最重要的使用之一是，使用 MapReduce 全部重写了产品索引系统（这个索引系统用来生成用于 Google 网页搜索服务的数据结构）。索引系统将我们的爬行系统检索到的大量文档作为输入，存储为一组 GFS 文件。 这些文档的原始内容超过 20 TB 的数据。 索引过程是以五到十次 MapReduce 操作的顺序运行。 使用 MapReduce（而不是先前版本的索引系统中专门的分布式传递）能够带来以下几个好处：

* 索引代码更简单、更轻量并且更易于理解，因为处理容错、分布式和并行的代码都隐藏在了 MapReduce 库中。例如，使用 MapReduce 后，计算的一个阶段的代码量从大约 3800 行 C++ 代码减少至大约 700 行。
* MapReduce 库的性能足够好，我们可以将概念上不相关的计算分开，而不是将它们混合在一起以避免额外的数据传递。 这使得更改索引过程变得容易。 例如，在我们的旧索引系统中需要几个月才能完成的一项更改在新系统中只需要几天时间就可以实现。
* 索引处理变得更容易操作，因为大部分由机器故障、运行缓慢的机器和网络问题（原文是 networking hiccups，直译是“网络打嗝”，更生动一些）导致的问题都由 MapReduce 库处理完了，而不需要操作员干预。此外，提升索引处理的性能也变得容易多了，直接给索引集群添加一些新机器就好了。









## 7. 相关工作

许多系统提供了受限的编程模型并使用这些限制来自动并行化计算。例如，可以使用并行前缀计算在 N 个处理器上以 log N 时间在 N 元素数组的所有前缀上计算关联函数。MapReduce 可以被视为基于我们在大型现实世界计算中的经验对其中一些模型的简化和提炼。 更重要的是，我们提供了可扩展到数千个处理器的容错实现。相比之下，大多数并行处理系统仅在较小规模上实现，并将处理机器故障的细节留给程序员。

批量同步编程和一些 MPI 原语提供了更高级别的抽象，使程序员更容易编写并行程序。这些系统与 MapReduce 的一个关键的不同是，MapReduce 利用受限编程模型来自动并行化用户程序并提供透明的容错。

我们的本地优化从诸如活动磁盘之类的技术中汲取灵感，其中计算被推送到靠近本地磁盘的处理元素中，以减少通过 I/O 子系统或网络发送的数据量。 我们在少量磁盘直接连接的商用处理器上运行，而不是直接在磁盘控制器处理器上运行，但一般方法是相似的。

我们的候补任务机制类似于夏洛特(Charlotte)系统中采用的急切调度机制。简单的急切调度的缺点之一是，如果给定的任务导致重复失败，则整个计算无法完成。我们使用跳过不良记录的机制修复了此问题的一些实例。

MapReduce 实现依赖于内部集群管理系统，该系统负责在大量共享机器上分发和运行用户任务。尽管不是本文的重点，但集群管理系统在思想上与 Condor 等其他系统相似。

作为 MapReduce 库一部分的排序工具在操作上类似于 NOW-Sort。源机器（map workers）对要排序的数据进行分区，并将其发送给 *R* 个 reduce worker 之一。每个 reduce worker 在本地（如果可能，在内存中）对其数据进行排序。当然 NOW-Sort 没有用户可定义的 Map 和 Reduce 函数，使我们的库广泛适用。

River 提供了一种编程模型，其中进程通过在分布式队列上发送数据来相互通信。与 MapReduce 一样，即使存在由异构硬件或系统扰动引入的非均匀性，River 系统也试图提供良好的平均情况性能。River 通过仔细安排磁盘和网络传输以实现平衡的完成时间来实现这一点。MapReduce 有不同的方法。通过限制编程模型，MapReduce 框架能够将问题划分为大量细粒度的任务。这些任务在可用的工作人员上动态安排，以便更快的工作人员处理更多任务。受限编程模型还允许我们在作业即将结束时安排任务的冗余执行，这在存在不均匀性（例如缓慢或卡住的工人）的情况下大大减少了完成时间。

BAD-FS 具有与 MapReduce 截然不同的编程模型，并且与 MapReduce 不同，它的目标是跨广域网执行作业。然而，有两个基本的相似之处。 (1) 两个系统都使用冗余执行来从故障导致的数据丢失中恢复。(2) 两者都使用位置感知调度来减少通过拥塞的网络链接发送的数据量。

TACC 是一个旨在简化高可用网络服务构建的系统。与 MapReduce 一样，它依赖重新执行作为实现容错的机制。

## 8. 总结

MapReduce 编程模型已在 Google 成功用于许多不同的目的。 我们将这一成功归因于几个原因。 首先，该模型易于使用，即使对于没有并行和分布式系统经验的程序员也是如此，因为它隐藏了并行化、容错、局部优化和负载平衡的细节。 其次，各种各样的问题很容易表达为 MapReduce 计算。例如，MapReduce 用于为 Google 的生产网络搜索服务、排序、数据挖掘、机器学习和许多其他系统生成数据。第三，我们开发了 MapReduce 的实现，它可以扩展到包含数千台机器的大型机器集群。该实现有效地利用了这些机器资源，因此适用于 Google 遇到的许多大型计算问题。

我们从这项工作中学到了一些东西。 首先，限制编程模型可以很容易地并行化和分布计算并使此类计算具有容错性。 其次，网络带宽是一种稀缺资源。 因此，我们系统中的许多优化旨在减少通过网络发送的数据量：本地优化允许我们从本地磁盘读取数据，将中间数据的单个副本写入本地磁盘可以节省网络带宽。 第三，冗余执行可用于减少慢速机器的影响，并处理机器故障和数据丢失。
