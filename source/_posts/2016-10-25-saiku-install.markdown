---
layout: post
title: "saiku安装及基础配置"
date: 2016-10-25 11:56:21 +0800
comments: true
categories: [Saiku, Data] 
keywords: 
description: 
---

####背景

saiku是一个开源的OLAP分析平台，在我们的项目中逐渐发现，除了基础的数据报表需求，以及数据挖掘需求外，也需要考虑把部分查询交互起来，一方面可以减少报表方面的压力，另外一方面也可以快速对数据进行验证，毕竟通过界面拖拽的方法相比直接写SQL还是要方便很多的。
在公司的另外一个项目里，saiku已经有比较成熟的应用，因此我们选择saiku作为我们需求处理的工具（尽管客观说saiku社区版还是有自己的问题的）。

####文档地址

	http://wiki.meteorite.bi/display/SAIK/Saiku+Features

####下载

通过wget下载最新的saiku社区版，目前版本是3.8.8：

	wget www.meteorite.bi/downloads/saiku-latest.zip

####启动安装注册
下载后进入saiku-server目录，通过./start-saiku.sh可以启动saiku（同理，通过./stop-saiku.sh关闭saiku）。如果正常，启动后通过你机器的8080端口 (浏览器里visit http://your_IP:8080)，可以访问到的saiku的登陆页面。

默认情况下，saiku的管理员账户和密码均是admin，但直接登陆后，会提示：

	Could not find license, please get a free license from http://licensing.meteorite.bi. You can upload it at http://server:8080/upload.html

这种情况是因为目前即使是社区版，也需要在saiku上注册：

	http://licensing.meteorite.bi/licenses
访问上述网址，可能会提示用户登陆，需要先注册用户。
登陆后，可以看到：

	LICENCE
		Create new licence
		List all licences
	COMPANY
		Create new company
		List all companies
这样的目录结构，我的情况是，需要先创建一个company，然后再创建licence （company在licence中是必选的），licence的hostname就填写你机器的IP（我没有试过其它是否可以），下载licence文件到本地，然后再通过http://your_IP:8080/upload.html将这个licence文件提交上去，整个注册过程就完成了。
注册完成后，通过admin/admin可以登陆进入saiku。

####端口设置
默认情况下，saiku启动在8080端口，如果需要更改，可以修改saiku-server/tomcat/conf/server.xml里的port配置，比如把8080全部替换为9090。

####权限控制
saiku server的权限控制主要通过tomcat设置：

	tomcat/webapps/saiku/WEB-INF
其中applicationContext-saiku.xml会引用applicationContext-spring-security.xml，后者默认会采用jdbc设置，在applicationContext-spring-security-jdbc.xml中配置dataSource：

	      <bean id="dataSource"                class="org.springframework.jdbc.datasource.DriverManagerDataSource">
                <property name="driverClassName" value="com.mysql.jdbc.Driver" />
                <property name="url"                        value="jdbc:mysql://your_IP:3306/admin" />
                <property name="username" value="your_name" />
                <property name="password" value="your_password" />
        </bean>

####数据源连接
在saiku中，我们需要通过预定义的Cube来指定数据访问。Cube是OLAP中的一个逻辑概念，通过对物理数据表的描述，将数据划分为dimension和metric（在saiku中为measure）。saiku中的Cube通过xml描述，具体含义可以参考[这里](http://mondrian.pentaho.com/documentation/schema.php#Cubes_and_dimensions)，后续有机会我会详细介绍。
因此，假设我们已经建立好Cube对应的xml，以及数据库中对应的fact表和dimension表，那么我们还需要告诉saiku如何找到它们。
通过在tomcat/webapps/saiku/WEB-INF/classes/saiku-datasources中新建文件配置：

	type=OLAP
	name=xxx
	driver=mondrian.olap4j.MondrianOlap4jDriver
	location=jdbc:mondrian:Jdbc=Jdbc:mysql://xxx/dbname;Catalog=<your_xml_path>

其中type一般是OLAP，name表示你Cube的名词，driver选择[mondrian](http://community.pentaho.com/projects/mondrian/)，它是saiku内部使用的数据处理引擎，支持MDX查询 （从这个角度说，saiku更像是查询平台），location里分别指定xml位置和mysql连接方式。

配置完成后，重启saiku，你需要的数据已导入。