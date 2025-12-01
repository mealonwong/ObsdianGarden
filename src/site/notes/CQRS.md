---
{"dg-publish":true,"permalink":"/cqrs/"}
---


CQRS（Command and Query Responsibility Segregation，命令和查询职责分离）


CQS（命令查询分离）设计模式建议将对象的方法映射到两类：方法要么改变对象的内部状态，但不返回任何内容，要么只返回元数据。这种方法称为**Command**。或者一个方法返回信息但不改变内部状态。这种方法称为**Query**。