mybatis有一级缓存和二级缓存

## 一级缓存

一级缓存是默认开启的，是sqlSession级别的缓存，在通过同一个sqlSession发出查询请求时，会走缓存。



## 二级缓存

默认是关闭的。

二级缓存是在一级缓存失效时，将缓存加载到二级缓存中，是namespace级别，即Mapper接口级别的缓存。