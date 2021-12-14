---
title: MapReduce 论文阅读笔记
date: 2021-12-14 14:13:36
updated: 2021-12-14 14:13:36
categories: [论文阅读笔记]
tags: [MapReduce,论文,分布式]
toc: true
---



[MapReduce 论文原文](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/16cb30b4b92fd4989b8619a61752a2387c6dd474.pdf)





## 1. 什么是 MapReduce





MapReduce 是一个编程模型，是一个为了处理与生成大数据集的相关实现。用户指定一个 `map` 函数，处理一个 key/value 键值对，以生成一系列中间 key/value 对，用户再指定一个 `reduce` 函数，合并与所有同一中间 key 的所有中间 value。很多真实世界的任务都可以以这个模型表达出来，论文中就要说这件事。

> 以我的理解来简单地说：MapReduce 编程模型就是把一个大的问题划分成很多小的问题，小的问题放在不同的机器上求解，然后再汇总到一起，就是最终我们要的结果。
>
> map 就是处理小问题的函数，reduce 就是用来汇总结果的函数。
>
> 例如 2.1 中会给出一个示例，要求在大量文章中统计每个单词出现次数。对于一个特定的单词来说，map 就是在单个文章统计其出现的次数，reduce 就是把 map 算出的这个单词在每个文章中出现的次数加起来，就是这个单词最终的计数。



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





































