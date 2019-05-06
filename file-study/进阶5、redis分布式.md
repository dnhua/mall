# Redis分布式算法原理
## 普通hash算法
通过取余数的方法来选取redis服务器。当新增节点或者删除节点时就不行了。

## 一致性hash算法
1. 环形hash空间
2. 把对象映射到hash空间上
3. 把cache映射到hash空间，使用相同的hash算法
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/chash1.PNG" width=50% height=50%/>
4. 移除添加cache影响较小
## 一致性hash算法的改进
>> 有可能cache节点分布不均匀，hash倾斜性
1. 引入虚拟节点
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/chash2.PNG" width=50% height=50%/>
2. 对虚拟节点再hash
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/chash3.PNG" width=50% height=50%/>
3. 命中率计算：(1-n/(n+m))*100%

# Redis分布式实战
## Redis分布式环境配置
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/redis1.PNG" width=50% height=50%/>

## 实战
1. 启动两个redis
2. 编写RedisShardedPool.java
3. 编写RedisShardedPoolUtil.java类，替换相应的代码