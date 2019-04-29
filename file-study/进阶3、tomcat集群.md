# tomcat集群
>> 提高服务的性能，并发能力，以及高可用性
>> 提供项目架构的横向扩展能力
# 一、tomcat集群的实现原理
通过Nginex负载均衡进行请求转发
## 初版架构和第二版架构对比
### 初版架构
![v1](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/v1.PNG)
### tomcat集群带来的挑战
1. Session登陆信息存储以及读取的问题
2. 服务器定时任务并发的问题
### 解决方案
1. 采用nginx ip hash policy
优点：可以不改变现有技术架构
缺点：<br>
a. 导致服务器请求(负载)不平均(完全依赖ip hash的结果)
<br>b. 在ip变化的环境下无法服务
### 二期架构
![v2](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/v2.PNG)

# 二、tomcat单机部署多应用
1. 修改/etc/profile增加tomcat环境变量
![tomcat1](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/tomcat1.PNG)
2. 第一个tomcat不变
3. 打开第二个tomcat目录下bin下catalina.sh，找到 # OS specific support. $var _must_be set to either true or false
4. 在刚刚找到的那个位置编辑，新增配置，并保存退出
<br>export CATALINA_BASE=$CATALINA_2_BASE
<br>export CATALINA_HOME=$CATALINA_2_HOME
5. 打开第二个tomcat的conf目录下server.xml，三个端口需要修改
![tomcat2](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/tomcat2.PNG)
![tomcat3](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/tomcat3.PNG)
![tomcat4](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/tomcat4.PNG)
