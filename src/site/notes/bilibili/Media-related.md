---
{"dg-publish":true,"permalink":"/bilibili/media-related/"}
---


### 媒资实体融合问题

#### 数据清洗对齐
1. 不同数据源数据格式不一样，不要将其转换为外部媒资库数据格式，但以bangumi为例，其采用kv（infoBox）形式存储数据，同一字段可能使用不同key，且v就是一串没有固定格式的字符串，因此数据清洗对齐比较繁琐，而且可能会遗漏部分数据或者解析错误。
2. 对于数据清洗需要着重注意一点，数据可以缺失，但不能错误。

#### 实体对齐
这个的重点在于如何判断两个媒资是同一个媒资？两个媒体人是同一个人？
1. 当前是基于规则判断，例如媒资=标题+导演，媒体人=名字+出生日+出生地区，规则严苛但收效甚微，甚至覆盖率不高的同时准确率也不尽人意。
	1. 首先由于部分数据缺失，例如某个媒资实体中缺少导演属性，那它自然无法与任何实体对齐匹配；或者是人的生日缺失，出生地格式不同也会导致影人实体无法对齐；
	2. 有时媒资名称或者影人名称的细微差别也可能导致匹配失败，例如莱昂纳多·迪卡普里奥和李昂纳多·迪卡普里奥，莱昂纳多迪卡普里奥
2. 关于格式问题可以通过相似度算法解决，例如增加一些属性集合，根据属性分配权重，根据相似度模型最终判断是否属于同一实体（采用余弦相似度判断文本相似度，或者Jaccard相似度判断属性集合相似度）
3. 如何从数据库中拿到可能相似的实体数据？
	1. 使用标题进行精确匹配或者模糊匹配？
4. 属性缺失问题如何解决？结合大模型是否能够解决这个问题？