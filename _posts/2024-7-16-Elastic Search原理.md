## 1、简介

- Elasticsearch是⼀个基于ApacheLucene库实现的，Restful⻛格的，分布式搜索和数据分析引擎。基 于倒排索引技术，实现了⾼性能的全⽂检索和数据分析功能。[官方网站](https://www.elastic.co/cn/elasticsearch/)

## 2、架构

### 2.1、倒排索引

- 将文本分词后获得词项，建立文本id和词项对应关系，是多对多的关系

  ![](https://github.com/Peweho/Peweho.github.io/raw/master/images/2024-7-16-Elastic Search原理/Snipaste_2024-07-16_10-29-58.png)

- 查询时间复杂度O(n)，排序后为O(logn)

### 2.2、Term Index

由于词项太多，放进内存占用会比较大，将词项建立前缀树，叶子结点指向词项在磁盘中的位置，减小了内存的占用、加快搜索速度

![](https://github.com/Peweho/Peweho.github.io/raw/master/images/2024-7-16-Elastic Search原理/Snipaste_2024-07-16_10-34-48.png)

### 2.3、Doc Values

- 当要对查询出的数据进行排序时，必须从倒排索引找到文档Id，再从**Sorted Field**拿出文档并根据字段进行排序。
- 为了提升效率，直接建立文档id和排序字段的索引，直接拿到排序后的文档id，这样结构式**Doc Values**

![](https://github.com/Peweho/Peweho.github.io/raw/master/images/2024-7-16-Elastic Search原理/Snipaste_2024-07-16_10-46-56.png)

### 2.4、Segment

具备完整搜索能的最小单元，包含Inverted Index、Term Index、Sorted Filed、DocValues

### 2.5、Lucene

- 单机搜索引擎库
- 由于文档会不断添加，防止对同一个Segment读写出现并发问题，规定只能读Segment，新的文档将建立新的Segment
- 搜索时并发读取全部Segment
- 定期进行Segment合并，减少Segment数量

### 2.6、Elastic Search

- 底层使用Lucene的基础上进行高性能、高可用、高扩展性支持持久化的搜索引擎
- 对外提供HTTP接口进行增删改查
- 搜索时分为查询和获取阶段
  - 查询阶段会使用倒排索引和Doc Values获得id
  - 获取阶段再从Sorted Field中获取文档

## 3、参考

- [视频资料](https://www.bilibili.com/video/BV1yb421J7oX?vd_source=1f9ea29590303cada0b4e521e1e038e0)

