---
author: "fishwings"                                                                     
date: 2022-08-19                                                                        
title: "Hyperspace: The Indexing Subsystem of Azure Synapse"                                                    
tags: [                                                                                 
    "大数据","论文","索引"                                                                              
]                                                                                       
categories: [                                                                           
    "索引",  
    
]      
---


# Hyperspace: The Indexing Subsystem of Azure Synapse

# 1. What

Hyperspace允许用户在其数据上构建多种类型的二级索引，通过多用户并发模型维护它们，并自动利用它们(无需更改应用程序代码)来实现查询/工作负载加速。

Hyperspace enables users to build multiple types of secondary indexes on their data, maintain them through a multi-user concurrency model, and leverage them automatically—without any change to their application code— for query/workload acceleration.

这个索引时覆盖索引，又可以叫派生数据集。

This index is covering index ,named derived datasets. 

# 2. why

通过标准行业基准测试和实际客户工作负载进行的评估表明，Hyperspace可以将查询执行速度提高10倍，在某些实际工作负载中，甚至可以提高两个数量级。

Evaluations over standard industry benchmarks and real customer workloads show that Hyperspace can accelerate query execution by up to 10x and in certain real-world workloads, even up to two orders of magnitude.

![Untitled](Hyperspace%20The%20Indexing%20Subsystem%20of%20Azure%20Synapse%200f5aaf83f50545e7b458d10ff6d96858/Untitled.png)

显示三个工作负载中每个查询所获得的收益。总体而言，当底层数据集采用CSV格式时，我们观察到TPC-H、TPC-DS和Customer的收益分别为75.0%、50.6%和89.9%，而当底层基本表数据集采用Parquet格式时，TPC-H、TPC-DS和Customer的收益分别为39.8%、50.5%和43.6%。

Shows the gain achieved for each query from the three workloads. Overall, we observe gains of 75.0% for TPC-H, 50.6% for TPC-DS, and 89.9% for Customer when the underlying datasets are in CSV format, and 39.8% for TPC-H, 50.5% for TPC-DS, and 43.6% for Customer when the underlying base table datasets are in Parquet format.

![Untitled](Hyperspace%20The%20Indexing%20Subsystem%20of%20Azure%20Synapse%200f5aaf83f50545e7b458d10ff6d96858/Untitled%201.png)

# 3. how

**underlying design**

1) hash - partitioned & (optional) Sorted:根据用户提供的索引配置，按每个分区中的索引列对索引进行hash分区和排序。索引列可以保存泛型数据类型，例如bloom过滤器。这允许索引扫描减少使用引用索引列的相等谓词(即点查找)的过滤器访问的数据量。如果索引列匹配连接键，并且优化器选择使用基于打乱的连接(例如，散列连接或排序-合并连接)，那么可以避免连接的打乱阶段，因为索引是预分区的。

Hash-Partitioned & (Optionally) Sorted: Index is hash partitioned and sorted by the index columns within each individual partition, based on user-provided index configuration. Index columns can hold generic data types, e.g., bloom filters. This allows index scans to reduce the amount of data to be accessed for filter with equality predicates (i.e., point look-ups) that reference the indexed columns. If the indexed columns match join keys and the optimizer chooses to use a shuffle-based join (e.g., a hash join or a sort-merge join), then the shuffle stage of the join can be avoided due to the index being pre-partitioned

![Untitled](Hyperspace%20The%20Indexing%20Subsystem%20of%20Azure%20Synapse%200f5aaf83f50545e7b458d10ff6d96858/Untitled%202.png)

Such indexes allow the following key query optimizations:

将表扫描替换为索引扫描。当索引包含作为关键列(也就是“索引列”)和数据/有效负载列(也就是“包含列”)的一部分来解析查询所需的所有信息时——覆盖索引——它们对于“仅索引”访问路径(包括优化连接)非常有用;见图1 (b)。例如，在一个或多个连接具有适当的分区对齐索引的情况下，可以完全消除全局shuffle(数据湖[54]中一些最昂贵的操作)。我们通过一项实证研究来说明这些好处，该研究衡量了在以下两种情况下避免数据洗牌的好处:(i)查询引擎知道连接中涉及的两个表的底层数据分布，因此它可以避免全局分布式洗牌;(ii)查询引擎不知道底层数据分布，因此它将执行完全的分布式洗牌。在(i)和(ii)两种情况下，我们将查询的执行时间与以下微基准查询进行比较:

**Replace Table Scans with Index Scans.** When the index contains all information required to resolve a query as part of its key columns (a.k.a., “indexed columns”) and data/payload columns (a.k.a, “included columns”)—a covering index—they can be tremendously useful for “index-only” access paths (including for optimizing joins); see Figure 1(b). For instance, in cases where one or more joins have appropriate partition-aligned indexes, global shuffles (some of the most expensive operations in data lakes [54]) can be entirely eliminated. We illustrate these benefits through an empirical study that we conducted to measure the benefits of avoiding data shuffling under two scenarios: (i) query engine is aware of the underlying data distributions of the two tables involved in a join so it can avoid a global distributed shuffle, and (ii) query engine is not aware of the underlying data distributions so it will perform a full distributed shuffle. We compare the query execution time as the number of CPU cores grow, for both cases (i) and (ii) with the following microbenchmark query:

![Untitled](Hyperspace%20The%20Indexing%20Subsystem%20of%20Azure%20Synapse%200f5aaf83f50545e7b458d10ff6d96858/Untitled%203.png)

这里lineitem和orders是1TB TPC-H基准[10]中最大的两个表。注意，这两种情况在查询执行时间上的差异高达2.5倍。随着洗牌数量的增加，这种差异将显著增加;

Here lineitem and orders are the two largest tables in the 1TB TPC-H benchmark [10]. Notice the difference of up to 2.5x in query execution time between the two cases. This difference will increase significantly as the amount of shuffling increases;

使用索引查找减少表扫描。当索引不包含给定查询时，它对于减少表扫描访问的数据量仍然有用，如图1(c)所示。在查询优化期间，Hyperspace使用这样的索引来基于列选择性消除表中可以跳过的部分。

注意。在实践中，很容易将其他类型的索引包含到多维空间中，只要它们可以用于索引扫描或查找。事实上，基于这一观点，我们正在将其他一些知名的数据结构，如物化视图、Z-ORDER、数据跳过、草图、倒排列表甚至bloom过滤器，嵌入到Hyperspace中。一个明显的挑战是如何使用这些索引来加速查询处理，这意味着当有这些索引可用时，查询优化器需要能够重写查询计划，并相应地估计它们的执行成本;

**Reduce Table Scans with Index Seeks**. When the index does not cover the given query, it may still be useful for reducing the amount of data accessed by a table scan, as shown in Figure 1(c). During query optimization, Hyperspace uses such indexes to eliminate portions of the table that can be skipped, based on column selectivity.

Remark. In practice, it is easy to include other types of indexes into Hyperspace as long as they can be used for index scans or seeks.Indeed, with this point of view, we are embedding several other well-known data structures, such as materialized views, Z-ORDER, data skipping, sketches, inverted lists and even bloom filters, into Hyperspace [43]. One obvious challenge is how to use these indexes to accelerate query processing, which implies that the query optimizer needs to be able to rewrite the query plan when such indexes are available and estimate their execution cost accordingly; 

# 4 CONCLUSION

Hyperspace不仅提供了查询性能加速，而且提供了一个自动化的生命周期管理框架

Hyperspace offers not only query performance acceleration but also an automated lifecycle management framework