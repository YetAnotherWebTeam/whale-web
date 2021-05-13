# 理解

1. 获知需求（倒推，用户管理+赛事管理）
   - 扩展用户属性，增加增删改查接口及文档；
   - 扩展赛事属性，编对应schema，增加增删改查接口及文档；
2. Code：Django现有接口（[Django]( )）+Model+后端DB
3. 熟悉Code，熟悉架构，增加功能



# 用户管理-User

## 接口

1. register 注册
   - post
   - email + passwd
   - 200{data:User, code:0}
2. login 登入
   - get
   - 200{data:User, code:0}
3. logout 登出
   - post
4. me
   - get
   - 200{data:User, code:0}
5. send-verification
   - post
   - email+passwd
   - 200{data:email, code:0}
6. verify-token
   - get
   - 200{data:email, code:0}
7. user-query 查询用户信息
   - get
   - user_id
   - 200:{data:User, code:0}
8. user-update 更新用户信息
   - post
   - User表内属性
   - 200:{data:User, code:0}
9. user-del 删除用户
   - delete
   - user_id
   - 200:{data:{}, code:0}
10. auth-group-quary 查看组信息
    - get
    - group_id
    - 200:{data:Group, code:0}
11. auth-group-add 增加组成员
    - post
    - group_id, user_id
    - 200:{data:Group, code:0}
12. auth-group-update 更改组成员信息
    - post
    - group_id, user_id，user_name
    - 200:{data:Group, code:0}
13. auth-group-del 删除组成员
    - delete
    - group_id, user_id
    - 200:{data:{}, code:0}

## User属性

增加 头像、学校、专业、公司、职位、权限

```
User:
      type: object
      properties:
        id:
          type: integer
          format: int64
          minimum: 1
        email:
          type: string
          format: email
        phone:
          type: string
        nickname:
          type: string
        date_joined:
          type: string
          format: date-time
        last_login:
          type: string
          format: date-time
        last_login_ip:
          type: string
          format: ipv4
        description:
          type: string
        groups:
          type: array
          items:
            $ref: "#/components/schemas/Group"
        picture：
        	type：string
        	format:picture
        college:
        	type: string
        majors:
        	type: string
        company:
        	type: string
        career:
        	type: stringS
        permissions:
        	type: strings
    Group:
      type: object
      properties:
        id:
          type: integer
          format: int64
          minimum: 1
        name:
          type: string
```





# 赛事管理-articles

## 接口

1. articles(add a article)
   - post
   - aritcle_id, title, content, status
   - 200:{data:Article, code:0}
2. articles/pk
   - get
   - article_id
   - 200:{data:Article, code:0}
3. article-query
   - get
   - article_id
   - 200:{data:Article, code:0}
4. article-update
   - post
   - Article 属性
   - 200:{data:Article, code:0}
5. article-del
   - get
   - article_id
   - 200:{data:{}, code:0}

## Articles属性

```
Article:
      type: object
      properties:
        id:
          type: string
        author:
          type: integer
        title:
          type: string
        content:
          type: string
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time
        status:
          type: string
```

