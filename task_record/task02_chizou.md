# 用户管理与文章增删改查接口设计

## 1.设计原则

参考：

https://zhuanlan.zhihu.com/p/31298060

https://editor.swagger.io/

### **参数传递**

遵循RESTful规范，使用了GET, POST, PUT, DELETE共4种请求方法。

1. GET：请求资源，返回资源对象
2. POST：新建资源，返回新生成的资源对象
3. PUT：新建/更新资源，返回完整的资源对象
4. DELETE：删除资源，返回body为空

- GET请求不允许有body， 所有参数通过拼接在URL之后传递，**所有的请求参数都要进行遵循[RFC 3986](https://link.zhihu.com/?target=https%3A//tools.ietf.org/html/rfc3986)的URL Encode**。
- DELETE删除单个资源时，资源标识通过path传递，批量删除时，通过在body中传递JSON。
- POST, PUT请求，所有参数通过JSON传递，*可选的请求参数，只传有值的，无值的不要传递*，contentType为application/json。

*4种请求动作中，GET、PUT、DELETE是幂等的；只有POST是非幂等的。幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。* **是非幂等是判断接口使用POST还是PUT的决定条件**

**注意： APP端获取json数据时，对于数值类型字段必须以数值类型转换，无论传递过来的值是否带引号。**



用post替代put

## 2.用户权限与信息管理

> 任务：
>
> 需要基于现有的用户属性，扩充表单中要求的额外属性，如头像、学校、专业等。
>
> 1. 编辑[openapi.yaml](./openapi.yaml)中`components.schemas.User`
> 2. 新增用户增删改查接口文档

### 接口设计

用户个人信息：

新增属性

头像/微信/学校/专业/公司/职位



超级管理员的用户权限管理：此处权限仅为文章编辑

可增加权限

tag：user-auth-manage

前缀：/api/v1/user-auth-manage

| url                             | 含义                 | 实现                                                         | 请求方法 |               请求参数 | 返回                              |
| ------------------------------- | -------------------- | ------------------------------------------------------------ | -------- | ---------------------: | --------------------------------- |
| /user-update                    | 用户修改个人信息     | 修改core_user                                                | post     |                   User | 200:{data:User, code:0}           |
| /group                          | 查看组内所有成员     | 查询auth_gropu                                               | get      |                     无 | 200:{data:Group，code：0}         |
| /group-detail/{group_id}        | 查看某个组成员       | 查询core_user_groups+core_user                               | get      |               group_id | 200:{data:[GroupUser]，code：0}   |
| /group-add                      | 组增加人员           | core_user_groups增加一条记录                                 | post     |      user_id，group_id | 200:{data:[GroupUser]表,code:0}   |
| /group-del/{user_id}/{group_id} | 从组内删除用户（将） | core_user_groups增加一条记录                                 | delete   |      user_id，group_id | 200:{data:~~[GroupUser]~~,code:0} |
| /edit-user                      | 修改某个用户权限     | 修改core_user_user_permissions+core_user_groups              | Post     | user_id，permission_id | 200:{data:UserAuth，code：0}      |
| /query-user                     | 查询某个用户的权限   | 从core_user_groups到auth_group_permissions + core_user_user_permissions | get      |      user_id，group_id | 200:{data:UserAuth，code：0}      |

### 超级管理员组管理

tag：group-auth-manage

前缀：/api/v1/group-auth-manage

| url  | 含义           | 实现 | 请求方法 | 请求参数 | 返回 |
| ---- | -------------- | ---- | -------- | -------- | ---- |
|      | 查询所有组     |      |          |          |      |
|      | 新增组         |      |          |          |      |
|      | 删除组         |      |          |          |      |
|      | 查看某个组权限 |      |          |          |      |
|      | 编辑某个组权限 |      |          |          |      |



### 关于权限与组的设计与数据表

组信息：auth_group

用户属于某些组：core_user_groups

用户拥有某些权限id: core_user_user_permissions

所有权限在一个表里：auth_permission

一个组可以拥有多个权限：auth_group_permissions

所以用户的权限：单独的权限id+所在组的所有权限



组：表示一种身份，比如教师，学生；正式员工，实习生

用户的权限显示：组+单独的，编辑的话，



### schemas数据格式

```json
User：{
	 +pic:string/url
   +wechat:string
   +school:string
   +profession:string
   +company:string
   +position:string
}

+GroupUser：{
	group_id
  [
  	email:string/email
  	user_name:string
  	user_id:integer/int64
  ]
}

+UserAuth：{
	user_id:integer/int64
  user_name:string
  [Group]
	[]

+Permission: {
  permission_id:integer/int64,
  perssion_name:string
}
}
```



## 3.文章管理

> 任务：
>
> 根据赛事管理的需求设计赛事管理相关的接口。

> 1. 确定赛事相关属性
>
>    增加一个领域属性：可选领域包括——赛事，干货，推荐，合作，活动
>
> 2. 编写对应schema
>
> 3. 编写增删改查接口文档

### 接口设计

每篇文章，属于某个主题（领域）

tag：article

前缀：/api/v1/article

| url                          | 含义                 | 实现                                                         | 请求方法 | 请求参数                 | 返回                            |
| ---------------------------- | -------------------- | ------------------------------------------------------------ | -------- | ------------------------ | ------------------------------- |
| /category                    | 查询所有主题         | 查询表blog_category                                          | Get      | 无                       | 200:{data:[Category],code:0}    |
| /article/{category}          | 根据主题查询所有文章 | 根据排序规则，查询blog_article下指定category_id的所有文章，默认显示前10篇 | get      | category_id              | 200:{data:[ArticleItem],code:0} |
| /article/{category}/{page}   | 根据主题查询分页文章 | 根据排序规则，查询blog_article下指定category_id的所有文章，显示：[(page-1)x10+1,page*10]（1-10，11-20）,page从1开始计数 | get      | category_id,current_page | 200:{data:[ArticleItem],code:0} |
| /article/{category}/{query}  | 根据文章标题查询文章 | 对blog_article的title模糊匹配                                | get      | category_id,query_desc   | 200:{data:ArticleItem,code:0}   |
| /article/delete/{article-id} | 删除某篇文章         | 根据id删除blog_article                                       | delete   | article_id               | 200:{data:delete sucess,code:0} |
| /article/create              | 增加一篇文章         | blog_article增加一条记录                                     | Post     | Article                  | 200:{data:Article, code:0}      |
| /article/edit                | 修改一篇文章         | blog_article修改一条指定记录                                 | Post     | Article                  | 200:{data:Article, code:0}      |

### schemas

```json
Article:{
	+theme：string
	+category_id：integer/int64
}
+Category：{
  id:integer/int64
  name:string
}
+ArticleItem:{
	img:string/url
  title:string
  article_id:integer/int64
  summary:string
  updateed_at:string/date-time
}
```

### 数据表

blog_article:+category_id(外链)/int

+blog_category：id/int,name/varchar(10)