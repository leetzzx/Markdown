# 运行环境
- Server version : Apache/2.4.41(Ubuntu)
- Php version : 7.4.3(cli)
- Php modules : php-mysql php-common
- libapache2-mod-php7.4
- Nodejs : v10.19.0
- mysql : Ver 8.0.25-0ubuntu0.20.04.1 for Linux on x86_64 ((Ubuntu))

# 相关环境的配置

## mysql配置
- 用户名 leetz 密码 @Ltzzx12138 设置为最强强度 `mysql> create user 'leetz'@'localhost' identified by '@Ltzzx12138';`
- 为其设置权限 `mysql> grant all privileges on *.* to 'leetz'@'localhost';`
- 刷新权限 `mysql> flush privileges;`
- 字符集编码为utf8mb4，是utf8超集，在此不做更改
- 同时更改用户的权限为可以远程登录`mysql> update user set host='%' where user='leetz';`
- 本来应该为普通用户设置写入操作，读取操作，以及更新权限，而不设置删除操作，但为了减少代码复杂度，这里一切从简，因此在管理员登录界面所用的密码，电话号码，也是普通用户表里面的

## 文件权限的编辑
```bash 
chmod 777 图片上传的文件夹
```
## apache2 配置
### 已经启动的模块
1. php-7.4



# 数据库初始化
```mysql
create database KSdb;
```
### 出现的问题
> 在这里出现了很严重的问题，在进行表的设计时，没有考虑到登录使用的名称应该是哪一个，虽然有一个主键，但并不适用与登录，因为是随机生成的，而且不好记，所以需要另外找个属性进行唯一性检验。但在将电话设置为unique的过程中，因为之前测试时插入了多个相同的电话号码，导致失败，因为黔驴技穷不知道怎么办，只好直接重新整理数据库

## 主表

### 1. user_info表
| 表属性       | 格式                                  | 另外的要求                                             |
|--------------|---------------------------------------|--------------------------------------------------------|
| 姓名         | char(16)                              | 编码格式使用UTF-8，需要对数据库存储类型进行设置        |
| 性别         | enum('男'，'女')                      | 必填                                                   |
| 电话号码     | char(11)                              | unique，必填，默认为null                               |
| 学校         | enum('郑州大学', '河南工业大学')      | 必填                                                   |
| 学号         | char(11)                              | 必填                                                   |
| **注册时间** | datetime 指账号注册时间               | 自动生成                                               |
| **uid**      | char(8)                               | 随机生成8位uid，作为用户表的主键                       |
| **设备类型** | char(10)                              | 必填                                                   |
| **个人风格** | char(20) 关联到另一个表上             | 是和其他表关联后的一个属性，从其他表获取               |
| #点赞总数    | int类型 关联到另一个表上              | 是动态的一个属性，暂不放到表中，具体实现放到html中实现 |
| 个人头像     | char(8) 和个人uid相关联的存储地址     | 地址在外部上传时自动转换为有规律的头像地址字符串       |
| **个人作品** | char(10) 关联到个人作品表用于展示作品 | 和个人上传作品表关联                                   |
| qq           | char(11)                              | 必填                                                   |
| email        | char(30)                              | 必填                                                   |
| 昵称         | char(16)                              | 必填，编码格式设置                                     |
| 用户类型     | enum('普通用户', '管理员', '摄影师')  | 在注册时进行用户类型的设置                             |
| 密码         | varchar类型                           | 个人密码设置，使用md5加密函数                          |
| （权限管理） | char类型                              | 进行默认设置                                           |
| 个人简介     | char类型                              | 默认设置，在用户个人管理界面提供单独的更改功能         |


```mysql
create table user_info (
	uid char(9) primary key,
	username char(16) not null,
	tele_num char(11) not null unique,
	gender enum('男', '女') not null default '男',
	register_time datetime not null,
	school enum('郑州大学', '河南工业大学') not null default '郑州大学',
	student_num char(12) not null,
	qq_num char(11) not null,
	email char(25) not null,
	nickname char(16) not null,
	avatar_path char(30) not null,
	password char(32) not null,
    user_type enum('普通用户', '摄影师') default '普通用户' not null,
	profile char(50) default '这个人太懒了，什么都没有写~\(≧▽≦)/~啦啦啦\)'
 );
 dev_type enum('type1', 'type2'),
 personal_type enum('风景', '人物'),
```

### 2. 用户作品表
| 表属性       | 格式                         | 另外的要求                 |
|--------------|------------------------------|----------------------------|
| 作品id       | char()                      | 作为用户作品表的主键       |
| 上传时间     | datetime                     | 自动生成                   |
| 点赞量       | int类型                      | 使用php与网页互动生成      |
| 作者         | char(8)                      | 与uid绑定，设立主键约束    |
| （评论）     | char(50)                     | 暂无，选择实现             |
| 作品名称     | char(8)                      | 在用户作品展示时显示       |
| 作品存储地址 | char(9)                      | 用于寻找作品显示           |
| 作品类型     | enum('风景', '人物', '实物') | 用于在不同的类型查询中实现 |
```mysql
create table user_work (
	pid char(9) primary key,
	upload_time datetime not null,
	w_thumbs integer default 0,
	author_id char(9),
	constraint fk_author_id foreign key(author_id) references user_info(uid),
	work_name char(16) not null,
	work_path char(30) not null,
	work_type char(8) not null
);
```

#### 附属关系表·用户关联表（选择实现）
| 表属性   | 格式     | 另外的要求 |
|----------|----------|------------|
| 关联id   | char(8)  | 作为主键   |
| 关注人   | uid      | 无         |
| 被关注人 | uid      | 无         |
| 关注时间 | datetime | 选择生成   |

```mysql
create table user_relation (
	rid char(20) primary key,
	follow_id char(8),
	followed_id char(8),
	follow_time datetime not null,
	constraint fk_follow_id foreign key(follow_id) references user_info(uid),
	constraint fk_followed_id foreign key(followed_id) references user_info(uid)
);
```

### 3. 用户评论表
| 表属性   | 格式                                   | 另外的要求   |
|----------|----------------------------------------|--------------|
| 评论id   | char(8)                                | 作为主键     |
| 评论人   | uid(8)                                 | 设立主键约束 |
| 被评论人 | uid(8)                                 | 设立主键约束 |
| 评论对象 | enum('图片', '评论的评论' ,'图集评论') | 暂无         |
| 评论时间 | datetime                               | 暂无         |
| 评论心情 | menu('悲伤', '伤心', '开心')           | 可选实现     |
```mysql
create table user_comment (
	cid char(30) primary key,
	comment_content char(50) not null,
	c_gid char(9),
	comment_id char(9),
	commented_id char(9),
	comment_type enum('comment', 'work') not null default 'work',
	comment_time datetime not null,
	c_thumbs integer default 0,
	c_dislike integer default 0,
	constraint fk_comment_id foreign key(comment_id) references user_info(uid),
	constraint fk_commented_id foreign key(commented_id) references user_work(author_id),
	constraint fk_c_gid foreign key(c_gid) references user_work(pid)
);
```
> 这里原来没有对comment_type设置default和not null属性，后来加上了，当前只做用户作品的评论，不对用户评论的评论做出适配，如果有精力，可以以后在进行适配，因此留下了这个属性。但是发现还是有问题，因为没有办法依据一个属性找到特定的评论进行布局，因此又添加了一列评论的作品的id，用来寻找相应的评论

### 4.摄影师申请表
| 表属性       | 格式     | 另外的要求                                                   |
|--------------|----------|--------------------------------------------------------------|
| rid          | char(9)  | 用户申请id                                                   |
| 真实姓名     | char(16) | 从用户表中获取                                               |
| 所用设备     | char(30) | 用户自己填写                                                 |
| 所有作品的赞 | integer  | 从作品表中获取（选择实现，可以后续维护的时候添加相应的功能） |
| 申请id       | char(9)  | 外键约束                                                     |
| 申请理由     | char(50) | 暂无                                                             |
| 申请时间     | datetime | 暂无                                                         |
> 在建表时没有考虑到对真实姓名的验证，在对用户进行申请的网页中，只是进行了简单的表单信息传递

```mysql
create table applications (
	aid char(9) primary key,
	realname char(16) not null,
	device char(30) not null,
	apply_id char(9),
	apply_reason char(50),
	apply_time datetime not null,
	constraint fk_apply_id foreign key(apply_id) references user_info(uid)
);
```

> 再次出现了忽视过的问题，没有对摄影师的信息另外做一个补充。因此直接把相应的一些信息另外又做了一个表进行存储，用于管理员的管理。
### 5. 摄影师商品表
| 表属性   | 格式     | 另外的要求   |
|----------|----------|--------------|
| 商品id   | gid      | 用作主键     |
| 拥有人   | uid      | 设立主键约束 |
| 上传时间 | datetime | 暂无         |
| 价格     | int类型  | 暂无         |
| 商品名称 | char(8)  | 暂无         |

> 应该注意的是所有商品的拥有人的uid所对应的user_type都是摄影师。
```mysql
create table cameraman_goods (
	gid char(9) primary key,
	owner_id char(9),
	upload_time datetime not null,
	price integer default 0 not null,
	good_name char(8) not null,
	good_description char(30) not null,
	constraint fk_owner_id foreign key(owner_id) references user_info(uid)
);

```
> 这里如果手动实现外键的话，需要对user_info数据库中的user_type 进行验证

### 6. 订单表
| 表属性     | 格式     | 另外的要求                                                                                                   |
|------------|----------|--------------------------------------------------------------------------------------------------------------|
| 订单号     | char(20) | 用作主键                                                                                                     |
| 订单人     | uid      | 设立主键约束                                                                                                 |
| 商品       | char(8)  | 在实现时由摄影师的商品表中获取                                                                               |
| 商品价格   | int类型  | 关联商品表获取相关信息                                                                                       |
| 下单时间   | datetime | 设置过期时间                                                                                                 |
| 商品拥有人 | uid      | 用于将来实现摄影师到底进行了多少的交易进行整理核算，设立主键约束，商品拥有人不应该与订单人相同，在此不做实现 |
| 下单人姓名 | char(16) | 暂无                                                                                                         |

> 这里同上还是没有考虑到搜索时的便利，因此在对订单进行管理的时候，只能通过商品名称进行管理

```mysql
create table order_info (
	oid char(20) primary key,
	order_good_name char(8),
	buyer_name char(16),
	buyer_id char(9),
	seller_id char(9),
	gprice integer not null,
	book_time datetime not null,
	constraint fk_seller_id foreign key(seller_id) references user_info(uid),
	constraint fk_buyer_id foreign key(buyer_id) references user_info(uid)
);
```
### 登录使用session登录
应该在所有的页面进行session的开启，同时还要进行各种页面的更改


### 


