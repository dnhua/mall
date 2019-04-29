# 一、maven快速入门
>> 实际开发时：可能不止一套环境，开发环境、测试环境、线上环境等，需要环境隔离。隔离环境之间各种配置存在的差异。
>> 隔离之后需要设置默认环境。<br>
>> 本地开发，环境（Local）<br>
>> 开发环境（Dev）<br>
>> 测试环境（Beta）<br>
>> 线上环境（Prod）

# 二、mmall环境分析
1. FTP服务器相关配置不一样
2. 数据库配置不一样
3. ... ...

## 2.1 maven 环境隔离解决的问题
1. 避免人工修改的弊端
2. 轻松分环境编译、打包、部署

## 2.2 环境隔离配置及原理
1. pom.xml中build节点增加
![maven_build](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/maven_build.PNG)
2. pom.xml中增加profiles节点
![maven_build](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/profiles.PNG)
3. 新建对应的文件夹，并把要隔离的文件分开，公共的留下
![maven_build](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/maven_dir.PNG)
4. import change
5. maven环境隔离编译打包命令
![maven_build](https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/maven_package.PNG)
