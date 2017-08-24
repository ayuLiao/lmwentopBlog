---
title: 实现RBAC权限控制
date: 2016-09-12 15:07:44
tags: PHP
---
## 背景：
前段时间自己找了些教程，动手写了一个简陋的商城，完成了商品模块和权限控制模块，打算花时间写几篇博文来总结一下自己在这次项目实践中得到的经验的，但是又被Android困了一些时间，晚了一些，现在总结一下自己商城后台RBAC的实现！

## 简介：
开发商城用的是ThinkPHP，所以RBAC的实现跟ThinkPHP有关，要有点TP基础

## 什么是RBAC
RBAC全称Role-Based Access Control，基于角色的访问控制，也就是用户通过角色和权限关联，程序判断用户拥有的角色是否拥有该权限。

## 用户、角色、权限之间的定义和关系
讲个简单的场景来理解用户、角色、权限的定义和关系。

假设现在商城后台有商品添加、商品删除、商品修改、会员添加、会员信息修改等权限，其实可以理解为商城的功能，我们是否拥有访问这些功能的权利就可以简单的理解为权限，那么我们又分出了商品管理员和会员管理员这两个角色，这两个角色分别拥有不同的功能，如果商品管理员拥有商品添加、商品删除、商品修改的权限，那么商品管理员这个角色就和这三个权限进行了关联，会员管理员可以拥有商品修改、会员添加、会员信息，不用奇怪，这个角色是可以拥有商品修改这个权限的，这个看开发人员怎么设置。
公司里有两个员工A、B，公司想让他们分别管理商品和会员，那么可以分出A用户和B用户，A用户关联商品管理员这个角色，B用户关联会员管理员这个角色，此时公司里来了个富二代，是老板的儿子，拥有后台所有的权限，那么就可以开个用户C，同时关联商品管理员和会员管理员，这样就拥有后台所有的权限了。

通过上面简单的场景可以比较直观的理解用户、角色、权限，用户就是我们具体的用户号，如用户C，用户号可以通过关联角色来获得该角色拥有的权限，如富二代关联的商品管理员和会员管理员，那么他就拥有商品添加、商品删除、商品修改、会员添加、会员信息修改这些权限，一般我们会开一个账号，拥有所有权限，包括用户的添加、删除和修改，通常叫他超级管理员或root。

那么他们之间的关系是什么？
通过上面的场景可以看出，都是多对多的关系，用户和角色多对多的关联关系，角色和权限多对多的关联关系，这样就通过角色将用户和权限关联在一起了。

## 实现RBAC
通过SQL建立权限表、角色表、用户表并进行关联，来初步实现RBAC。

权限表的SQL实现
```SQL
	DROP TABLE IF EXISTS ds_privilege;
	CREATE TABLE ds_privilege
	(
		id smallint unsigned not null auto_increment,
		pri_name varchar(30) not null comment '权限名称',
		module_name varchar(20) not null comment '模块名称',
		controller_name varchar(20) not null comment '控制器名称',
		action_name varchar(20) not null comment '方法名称',
		parent_id smallint unsigned not null default '0' comment '上级权限的ID，0：代表顶级权限',
		primary key(id)
	)engine=InnoDB default charset=utf8 comment '权限表';
```
ThinkPHP中通过模块名称、控制器名称和方法名称来构建一个具体的URL，这里所谓的权限就是打开后台相应模块的URL，你有权利访问这个URL，进入这个功能的页面，那么就可以说你拥有这个权限，所以这个权限表存入模块名、控制器名和方法名，当要应用时Thinkphp从该表中取出这些名词拼出相应的URL则可。

如图，可以看出这个商品列表的界面用来显示商品的信息
![Thinkphp拼URL](http://obfs4iize.bkt.clouddn.com/Thinkphp%E6%8B%BCURL.png)

如果将URL的访问权限给禁用，则该用户就没有查看商品详情页的权限了
![Thinkphp](http://obfs4iize.bkt.clouddn.com/Thinkphp%E6%97%A0%E6%9D%83%E8%AE%BF%E9%97%AE.png)

该表中的parent_id，用于无限流，所谓的无限流就是存放该表中的其他id，用于跟该表中的其他id建立关联，在该权限表中，权限是有所属的，如商品添加和删除的权限属于商品修改权限的下级权限，也就是拥有商品添加权限就一定拥有商品修改的权限，但是拥有商品修改的权限不一定有商品添加权限

![无限流](http://obfs4iize.bkt.clouddn.com/%E6%9D%83%E9%99%90%E6%97%A0%E9%99%90%E6%B5%81.png)

角色表的SQL实现
```SQL
	DROP TABLE IF EXISTS ds_role;
	CREATE TABLE ds_role
	(
		id smallint unsigned not null auto_increment,
		role_name varchar(30) not null comment '角色名称',
		primary key(id)
	)engine=InnoDB default charset=utf8 comment '角色表';
```
角色表就一个角色名词，用来连接用户和权限

建立一个中间表，管理角色和权限，这种建立中间表的做法在多对多的关系中非常常用
```SQL
	DROP TABLE IF EXISTS ds_role_privilege;
	CREATE TABLE ds_role_privilege
	(
		pri_id smallint unsigned not null comment '权限的ID',
		role_id smallint unsigned not null comment '角色的ID',
		key pri_id(pri_id),
		key role_id(role_id)
	)engine=InnoDB default charset=utf8 comment '角色权限表';
```
通过两个键来连接角色表和权限表

用户表的SQL实现
```SQL
	DROP TABLE IF EXTSTS ds_admin;
	CREATE TABLE IF NOT EXISTS ds_admin
	(
		id tinyint unsigned not null auto_increment,
		username varchar(30) not null comment '账号',
		password char(32) not null comment '密码',
		is_use tinyint unsigned not null default '1' comment '是否启用 1：启用0：禁用',
		primary key (id)
	)engine=InnoDB default charset=utf8;
```
用户表存用户的账号和密码，通过is_use来管理该用户账号是否可以使用

建立用户和角色的中间表来实现多对多的关系
```SQL
	DROP TABLE IF EXISTS ds_admin_role;
	CREATE TABLE ds_admin_role
	(
		admin_id tinyint unsigned not null comment '管理员的ID',
		role_id smallint unsigned not null comment '角色的ID',
		key admin_id(admin_id),
		key role_id(role_id)
	)engine=InnoDB default charset=utf8 comment '管理员角色';
```

在数据初始化是，添加一个超级用户，拥有一切权限
```SQL
	INSERT INTO ds_admin VALUES(1,'root','fd1b6774ff90e21143eefdc14d16896e',1);
```

## 中间表的作用
简单描述一下中间表的作用

角色表 role，拥有a、b两个角色
```
id    role_name
-------------
 1        a
 2        b
```

权限表 privilege，拥有a、b、c三个权限
```
id    pri_name
-------------
 1        a
 2        b
 3        c
 ```

a角色拥有bc两个权限
ds_role_privilege
```
role_id   pri_id
--------------------
   1        2              -->  1这个角色拥有2这个权限
   1        3              -->  1这个角色拥有3这个权限
```


如果要从上面的5张表中取出管理员ID为3的管理员的所有权限，其SQL的三种写法
```SQL
 SELECT * FROM ds_privilege WHERE id IN(
	SELECT pri_id FROM ds_role_privilege WHERE role_id IN (
		SELECT role_id FROM  ds_admin_role WHERE admin_id=3
	)
 )
```
用SELECT嵌套，先从ds_admin_role（用户和角色的关联表）取出该用户所对应的角色，再从ds_role_privilege（角色和权限关联表）取出角色关联的权限，在权限表中通过权限id找出对应的权限则可

写法二：
```SQL
 SELECT a.*
   FROM ds_privilege a,ds_role_privilege b,ds_admin_role c
    WHERE c.admin_id=3 AND b.pri_id=a.id AND b.role_id=c.role_id
```
给ds_privilege、ds_role_privilege、ds_admin_role分别取别名，再写出条件

写法三：
```SQL
 SELECT b.*
  FROM ds_role_privilege a
   LEFT JOIN ds_privilege b ON a.pri_id=b.id
   LEFT JOIN ds_admin_role c ON a.role_id=c.id
   WHERE c.admin_id=3
```
一般认为第三种写法效率最高

## 在ThinkPHP中具体运用
一般写在一个控制器A的构造方法中，该控制器继承Think中的Controller控制器，然后写其他控制器时，继承该控制器A，那么当其他控制器要开启时，必然要执行父控制器(A)中的构造方法中的代码，这些代码先于其他逻辑代码，所以在这里写用于权限判断代码十分合适

```PHP
	// 验证当前管理员是否有权限访问这个页面
	
	// 1. 先获取当前管理员将要访问的页面	 - TP带三个常量
	$url = MODULE_NAME .'/'. CONTROLLER_NAME .'/'. ACTION_NAME;

	// 查询数据库判断当前管理员有没有访问这个页面的权限
	$where = 'module_name="'.MODULE_NAME.'" AND controller_name="'.CONTROLLER_NAME.'" AND action_name="'.ACTION_NAME.'"';
```