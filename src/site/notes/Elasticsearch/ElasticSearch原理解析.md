---
{"dg-publish":true,"permalink":"/elasticsearch/elastic-search/"}
---


ElasticSearch底层基于Lucene，因此，要深入理解ElasticSearch，首先需要了解Lucene。  
Lucene  
倒排索引结构  
Lucene的核心就是倒排索引（Inverted Index），倒排索引是一种索引方法，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。  
Lucene会将文档分成一个一个的Term（单词），然后建立倒排索引。首先让我们先来看看倒排索引的结构：
![](https://pic.imgdb.cn/item/658e42d1c458853aeff0d6a9.webp)

Term Index：Lucene使用FST实现Term Index，Term Index是Term Dictionary的索引，可以快速查找一个Term是否在Dictionary中；并且能够快速定位Block的位置。  
Term Dictionary：Segment的字典  
Postings：记录了出现过的某个Term的文档列表及该Term在文档中出现的位置信息（Payload）  
Payload的作用：  

1. 存储每个文档都有的信息
2. 影响词的评分

从上图可以看出，Lucene使用[FST](https://link.zhihu.com/?target=https%3A//www.shenyanchao.cn/blog/2018/12/04/lucene-fst/)（Finite State Transducer）实现，FST类似字典树，但与字典树又有很大的不同，FST有如下特点：  

1. 共享前缀
2. 共享后缀
3. 确定
4. 无环
5. Transducer：接收特定的序列，终止于Final状态，同时会输出一个值

因此可以看出，FST相比于字典树又大大地减少了内存使用率。  
Lucene 文件结构
![](https://pic.imgdb.cn/item/658e4328c458853aeff1c062.webp)

**Lucene索引文档（Document）的流程：**  

1. Tokenizer：分词阶段，会将文档分成一个一个的词元（Token）
2. Linguistic Processor：语言处理阶段，转为小写或者将单词缩减为词根形式等，处理的结果成为词（Term）
3. Indexer：索引阶段，此阶段会创建字典、创建倒排表等等


**Lucene搜索文档（Document）的流程：**  

1. Lexical Analysis：词法分析，识别单词和关键字
2. Syntactic Analysis：语法分析，根据语法规则来生成一棵语法树
3. Linguistic Processor：语言处理，转为小写或者将单词缩减为词根形式等
4. Search Index：搜索索引，找出包含关键字的文档链表，根据语法树对文档链表进行合并操作，得到最终要搜索的文档
5. Order Result：结果排序，会计算权重，判断Term之间的关系，从而得到排序后的文档列表


**ElasticSearch索引流程：**  

1. 索引请求首先会发送到Client Node，Client Node会根据路由规则将请求转发到Data Node(Primary Shard)进行处理（Parimary Shard所在的Data Node从Master Node获取）
2. Primary Shard会往Lucene写，写成功后会再写到Translog（Translog是ElasticSearch为了提高可靠性引入的，因为Lucene首先会先写内存，如果宕机了数据就丢了），然后会将广播到Replica Shard进行处理
3. 在所有的Replica Shard都写成功后则返回给用户成功索引数据

ElasticSearch默认的路由策略是： `hash(routing) % num_primary_shards`  
routing的默认值是文档id
![](https://pic.imgdb.cn/item/658e44fdc458853aeff756bd.webp)


**ElasticSearch扩容：**

ElasticSearch默认的路由策略是： `hash(routing) % num_primary_shards`  
可以看出，路由策略是和Shard数量是强相关的。因此如果我们强制增加Shard的话会破坏路由规则。  
那么如果我们需要扩容怎么办呢？  
答案是我们可以重新创建一个索引，然后将老索引的数据迁移到新的索引上。ElasticSearch为了简化这个流程，提供了[Reindex API](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/5.6/docs-reindex.html)。
