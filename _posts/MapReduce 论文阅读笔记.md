---
title: MapReduce 论文阅读笔记
date: 2021-12-14 14:13:36
updated: 2021-12-14 14:13:36
categories: [论文阅读笔记]
tags: [MapReduce,论文,分布式]
toc: true
---



传送门: [MapReduce 论文原文](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/16cb30b4b92fd4989b8619a61752a2387c6dd474.pdf)

> 这里需要一个前置知识，你需要先了解一下函数式语言中的 map 和 reduce，因为 MapReduce 就是借鉴了函数式语言中的 map 和 reduce 的思想，如果你不了解这个，后面理解起来可能会有些困难。

> 这篇文章不是教学，是我自己的笔记，也许对你有用，也许只有我自己看得懂 =。= ！



## 1. 什么是 MapReduce



MapReduce 是一个编程模型，是一个为了处理与生成大数据集的相关实现。用户指定一个 `map` 函数，处理一个 key/value 键值对，以生成一系列中间 key/value 对，用户再指定一个 `reduce` 函数，合并与所有同一中间 key 的所有中间 value。很多真实世界的任务都可以以这个模型表达出来，论文中就要说这件事。



<!--more-->

以这种函数式风格编写的程序会被自动并行化，在大型计算机集群上执行。run-time 系统负责划分输入数据、规划程序跨一组机器执行、处理机器故障与管理跨机器交流的细节。这样就使得没有任何并行与分布式系统相关经验的程序员可以轻松的使用大型分布式系统资源。

MapReduce 的实现运行在一个大型计算机器群上，并且是高可伸缩的：一个典型的 MapReduce 计算是在数千台机器上处理数 TB 的数据。程序员会发现这个系统非常易用：已经有数百个实现好的 MapReduce 程序，并且每天有超过一千个 MapReduce 任务在 Google 的集群上执行。（这数据是 04 年论文上的，现在多的多了。）





## 2. 编程模型



计算程序拿到一组输入 key/value 对，然后产出一组输出 key/value 对。MapReduce 库的用户可以把这个计算表达为两个函数：*Map* 和 *Reduce*。

*Map*，由用户编写，获取一个输入 kv 对，然后生成一组中间 kv 对。MapReduce 库同一中间 key `I` 的所有中间 value 组合到一起，发送给 *Reduce* 函数。

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



-

这个示例完整的代码在论文最后的 Appendix A，这里直接在下面给出。

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



*Map* 调用通过自动地划分输入数据为一组 *M 个 split* 分布在多个机器上，这些输入 split 可以在不同的机器上并行处理。*Reduce* 调用分布式的，其通过使用划分函数（例如 *hash(key) mod R*）划分中间 key 空间为 R 个 piece 来实现，R 的值和划分函数都是由用户指定的。

下图展示了在我们的实现中，一个 MapReduce 操作的总体流程。

![Figure 1: Execution overview](https://gukaifeng.cn/posts/mapreduce-lun-wen-yue-du-bi-ji/MapReduce_Figure_1.png)

当我们的程序调用 *MapReduce* 函数时，会依次发生下面的事（图中的编号标签与下面的序号相对应）：

1. 用户程序中的 MapReduce 库先把输入文件划分为 M 个 split，每个 split 的大小一般是 16-64 MB（用户可以通过一个可选的参数控制）。然后 MapReduce 在一个机器集群上启动很多个程序的副本。
2. 这些程序的副本中有一个是特殊的，叫做 master。master 给其他的 workers 分配工作。一共有 *M* 个 map 任务和 *R* 和 reduce 任务要分配，master 挑选出闲置的 workers，给每个闲置的 worker 指派一个 map 任务或 reduce 任务。
3. 一个被指派了 map 任务的 worker 读对应的输入 split 的内容。其从输入数据中解析出 key/value 对，然后将每个 key/value 对传递给用户定义的 *Map* 函数，由 *Map* 函数生成的中间 key/value 对被缓存在内存中。
4. 内存中缓存的中间 key/value 对会被周期性的写入本地磁盘，并且会被划分函数划分为 *R* 个部分。然后这些中间 key/value 在磁盘上的位置会被传递回 master，master 再把这些位置信息传递给 reduce workers。
5. 当一个 reduce worker 收到了 master 提供的这些中间 key/value 对在磁盘上的位置信息，其就会使用远程过程调用，从 map workers 的本地磁盘中读取缓存的中间 key/value 对数据。当一个 reduce worker 读完了全部的中间数据，就会对这些中间数据按 key 排序，这样同一个 key 的所有数据就都放在了一起。排序是很有必要的，因为会有很多不同的 keys 映射到同一个 reduce 任务中。如果中间数据太大，无法全部放入内存，就会使用外部排序。
6. reduce worker 遍历排序好的中间数据，将遇到的每个不同的中间 key 与与之对应的中间 values 集合传递给用户定义的 *Reduce* 函数。*Reduce* 函数的输出被附加到一个这个 reduce 分块的最终输出文件中。
7. 当全部的 map 任务和 reduce 任务全部完成后，master 会唤起用户程序。此时，用户程序中的 *MapReduce* 调用结束，将继续执行用户程序中后面的代码。



当所有的操作全部成功完成时，mapreduce 执行的输出是可用的，且被划分进 *R* 个输出文件（每有一个 reduce 任务，就会有一个输出文件，文件名由用户指定）。

一般来说，用户不需要组合这 *R* 个输出文件为一个。用户常常会把这些输出文件作为输入传递给另一个 MapReduce 调用，或者在另一个分布式应用（可以处理划分进多个文件的输入的分布式应用）中使用这些输出文件。





### 3.2. Master 数据结构



### 3.3. 容错





























