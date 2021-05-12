# 用openapi设计restful风格接口

## 流程

操作openapi.yaml，设计接口

对于openapi.yaml的编辑，可以利用swagger editor（需要下载docker），或者vscode插件（搜索openapi，打开yaml文件，工具栏预览），或者在线编辑器https://editor.swagger.io/。

根据规范，联系业务设计接口。

通过编辑器测试接口及报文格式。

除此之外，我们还可以通过命令行工具`curl`或者应用[Postman](https://www.postman.com/)模拟HTTP请求进行验证。



将接口规范文档转成mock server，给前端用。

切换到client目录：

```
npm run mock
```

可以看到已经编写的接口。



## 测试

在swagger editor测试接口：

点击其中一个接口（如`/api/v1/me`），点击**Try it out** - **Execute**

完成接口规范文档的编写后，重启mock server。并可以通过运行下面命令，使前端项目以mock server为接口进行启动：

```
API_PORT=4010 npm run serve
```

## 接口设计

- `api/v1/verify/<token> [name='verify verification token']`

  ```python
  @api_view(['GET'])
  def verify_verification_token(request, token):
      email = validate_token(token)
      if not email:
          return Response({"data": None, "code": 400}, status=status.HTTP_400_BAD_REQUEST)
      return Response({"data": email, "code": 0})
  
  ```

  Get,传参数token（varcher（ 40）），验证邮箱。

  成功，返回200:{"data": email, "code": 0}

  失败，返回400:{"data": None, "code": 400}

  表里有一个authtoken_token:

  Key——varcher（ 40）

  Created——datetime（6）

  User_id——int

- `api/v1/register [name='register']`

  ```python
  @api_view(['POST'])
  def register(request):
      data = request.data
      email = validate_token(data.get('token'))
      if not email:
          return Response({"data": None, "code": 400, "message": "Invalid token"}, status=status.HTTP_400_BAD_REQUEST)
      password = data.get('password')
      if not password:
          return Response({"data": None, "code": 400, "message": "Password is required"}, status=status.HTTP_400_BAD_REQUEST)
      try:
          validate_password(password)
      except django.core.exceptions.ValidationError as e:
          logger.error(e)
          return Response({"data": None, "code": 400, "message": e.messages}, status.HTTP_400_BAD_REQUEST)
      user = UserModel.objects.filter(email=email).first()
      if user:
          return Response({"data": None, "code": 409, "message": "Email has been registered"}, status=status.HTTP_409_CONFLICT)
      user = UserModel(email=email, phone=None)
      user.set_password(password)
      user.save()
      django_login(request, user)
      return Response({"data": UserSerializer(user).data, "code": 0}, status=status.HTTP_201_CREATED)
  
  ```

  post，请求体data：

  token/email，password

  返回：

  token非法，400:{"data": None, "code": 400, "message": "Invalid token"}

  缺少密码：400{"data": None, "code": 400, "message": "Password is required"}

  密码原因：400{"data": None, "code": 400, "message": e.messages}

  邮箱占用：409{"data": None, "code": 409, "message": "Email has been registered"}

  新建用户：201{"data": UserSerializer(user).data, "code": 0}

  查看me的

  user{ 'id',

  ​            'email',

  ​            'phone',

  ​            'nickname',

  ​            'date_joined',

  ​            'last_login',

  ​            'last_login_ip',

  ​            'description',

  ​            'groups',}

- `api/v1/articles [name='articles']`

  ```python
  class ArticleListCreateView(BasicListCreateAPIView):
      permission_classes = [IsAuthenticated|ReadOnly]
      serializer_class = ArticleSerializer
      queryset = Article.objects.all()
  
      def create(self, request, *args, **kwargs):
          data = request.data
          data['id'] = shortuuid.uuid()
          serializer = self.get_serializer(data=data)
          serializer.is_valid(raise_exception=True)
          self.perform_create(serializer)
          headers = self.get_success_headers(serializer.data)
          return Response(
              {
                  'data': serializer.data,
                  'code': 0,
              },
              status=status.HTTP_201_CREATED, headers=headers
          )
  
      def perform_create(self, serializer):
          serializer.save(author=self.request.user)
  
  ```

  新建文章

  传：post，报文，Article.objects.all()

  返回：

  头：成功头，不知

  201 {

  ​                'data': serializer.data,

  ​                'code': 0,

  ​            },

表：blog_article

id——varchar(36)
title——varchar(64)
content——longtext
created_at——datetime(6)
updated_at——datetime(6)
status——int
author_id——int



```python
id = models.CharField(_('id'), primary_key=True, max_length=36)
    author = models.ForeignKey(User, on_delete=models.CASCADE, null=False)
    title = models.CharField(_('title'), max_length=64, blank=False)
    content = models.TextField(_('content'), blank=False)
    created_at = models.DateTimeField(_('created at'), auto_now_add=True)
    updated_at = models.DateTimeField(_('modified at'), auto_now=True)
    status = models.IntegerField(_('status'), choices=Status.choices, default=Status.normal)
```



- `api/v1/articles/<pk> [name='article']`

  查看某篇文章

  参数是pk/id

  返回：

  Response(

  200{

  ​            'data': response.data,

  ​            'code': 0,

  ​        }

查看pk赛事列表

标题，发起者

get：title，id，根据author_id到core_user获取nickname(author_name)

```python
class ArticleDetailView(BasicRetrieveUpdateDestroyAPIView):
    permission_classes = [IsAuthenticated|ReadOnly]
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

查看详情



## 问题解决

需要注意有两个接口需要提供参数。

接口里{}内放参数名称，在parameters:里的 name要和参数名称一致。不然会出现422错误。

最好有一个400返回，表示找不到。



测试完成，需要在编辑器和命令行都显示正确。

编辑器需要出现200。

命令行都是success

![image-20210512191808519.png](http://ww1.sinaimg.cn/large/a8adbf8cly1gqfv2vuonkj21a80kwe5a.jpg)



## openapi参考

https://help.coding.net/docs/management/api/import/openapi.html

```yaml
openapi: 3.0.0      # 代表使用 OpenAPI v3 规范
info:               # 这里写 API 文档基础信息
  title: 宠物商店 API 文档    # API 文档标题
  version: '1.0.0'          # API 文档版本
  description: 这是一篇关于宠物商店的 DEMO API 文档，仅做参考。        # API 文档描述
  contact:        # 联系信息
    name: CODING
    url: 'https://coding.net'
    email: support@coding.net
servers:      # 这里写 API 服务器地址，多条可代表不同环境
  - url: 'https://petstore.com/api/v1'
    description: 生产环境
  - url: 'http://test.petstore.com/api/v1'
    description: 测试环境
tags:    # 标签，CODING 中可作为分组依据，name 作为分组名称
  - name: 宠物
    description: 所有关于宠物的内容
  - name: 会话
    description: 关于用户的注册、登录和登出
paths:        # 这里写具体的 API 的相关信息（API 名称、请求类型、摘要、请求参数、响应等）
  /pets/{petId}:     # 第一层，API 路由
    get:            # 第二层，请求方法（Method）,如: get, post, delete 等
      tags:         # 标签，与上方全局的标签对应，将该接口归于对应 tag 的组中
        - 宠物
      summary: 宠物详情      # 接口摘要，CODING 中可作为接口标题
      operationId: get-pet   # 接口 ID（必填），接口的唯一识别方式 
      description: 获取宠物信息      # 接口描述信息
      parameters:            # 接口参数，可多条，支持路由、Query、Header 参数
          - schema:            # 参数结构
          type: string          # 参数类型，integer / string / array 等
          name: petId          # 参数 key
          in: path             # 路由（path）类型的参数
          required: true        # 是否必填
          description: 宠物 ID   # 参数描述
        - schema:
            type: string
            enum:         # 参数枚举
              - application/json
              - application/xml
            default: application/json
          in: header        # header 类型的参数
          name: Accept
          description: 返回数据媒体类型
        - schema:
            type: integer
            default: 1     # 参数默认值
          in: query       # Query 类型的参数
          name: page
          description: 页码
      security:             # 接口授权信息，此处与全局 components/securitySchemes 对应
        - petstore_auth:      # 此处与 components/securitySchemes/petstore_auth 对应
            - 'read:pets'      # 此处代表接口 OAuth Scope 信息
      responses:              # 接口响应信息
        '200':                 # 响应状态码
          description: 以数组形式返回宠物信息       # 响应描述信息
          headers:
            X-PAGE:
              schema:
                type: integer
              description: 当前页码
          content:           # 响应内容规范描述
            application/json:    # 响应内容 Media Type
              schema:          # 响应内容结构
                type: array
                description: 宠物信息
                items:
               # $ref 可用来引用 components/schemas 中的各种实体声明，减少相同数据结构冗余，增加复用性
                  $ref: '#/components/schemas/Pet'   
              examples:   # 响应范例，建议填写，可用于 CODING API 文档范例显示以及 Mock API 模拟数据
                请求成功范例:   # 范例名称
                  value:     # 范例，展示时将转换成 JSON 形式
                    - id: 1
                      name: doggie
        '400':
          description: 请求失败
        patch:          # patch 请求方法
            tags:
        - 宠物
      summary: 单独更新宠物属性
      operationId: patch-pet
      description: 单独更新宠物属性
      deprecated: true       # 接口是否已废弃
      requestBody:  # post / put / patch 等请求需要传输 body 时，请指定 requestBody，Path / Query / Header 请写在 parameters 中
        content:
          application/x-www-form-urlencoded:    # 媒体类型 Media Type
            schema:     # 参数结构与上述 parameters 中相同
              type: object
              properties:
                categoryId:
                  type: integer
                  description: 分类 ID
                status:
                  type: string
                  description: 状态
                  enum:
                    - available
                    - pending
                    - sold
                photoUrls:
                  type: array
                  description: 照片地址列表
                  items:
                    type: string
                    description: 照片地址
                name:
                  type: string
                  description: 宠物名称
                isHealthy:
                  type: boolean
                  description: 是否健康
                tagIds:
                  type: string
                  description: 标签 ID 数组
            examples:       # 请求 Body 范例，将在 CODING API 文档右侧范例区展示
              更新范例:       # 范例名称
                value:        # 范例内容，以下将以 JSON 形式展示
                  status: sold
      responses:
        '200':
          description: 更新成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
components:    # 通用组件，可定义复用的实体、响应、请求以及授权信息等
  schemas:     # 实体信息
    Pet:      # 实体 key
      title: Pet       # 名称
      type: object      # 实体数据结构类型
      description: 宠物实体信息           # 描述信息
      properties:            # 实体参数，可多条
        id:
          type: integer
          format: int64
          description: ID
        name:
          type: string
          example: doggie
          description: 名称
  securitySchemes:            # API 所使用的授权方式，可多条，CODING 会展示所有授权方式
    petstore_auth:     # 授权方式 Key
      type: oauth2      # 授权类型，可为：apiKey、http、oauth2、openIdConnect
      flows:       # OAuth2 所需要的流程信息
        authorizationCode:  # OAuth2 授权类型，可为：implicit、password、clientCredentials、authorizationCode
          authorizationUrl: 'https://petstore.com/oauth/authorize'    # 跳转授权地址
          tokenUrl: 'https://petstore.com/oauth/token?grant_type=authorization_code'    # 获取 token 地址
          scopes:      # 所有 scope 列表
            'write:pets': 修改宠物信息权限
            'read:pets': 查看宠物信息权限
          refreshUrl: 'https://petstore.com/oauth/token?grant_type=refresh_token' # 刷新 token 地址
      description: OAuth2 授权模式                    # 描述信息
```

## 设计

新增path

```yaml
 "/api/v1/send-verification":
    post:
      summary: send verification mail
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/SendVerificationForm"
      responses:
        '200':
          description: success logout
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: integer
                  code:
                    type: integer

  "/api/v1/verify/{token}":
    get:
      summary: verify verification token
      parameters:
      - name: token
        in: path
        description: token of user
        required: true
        schema:
          type: string
      responses:
        '200':
          description: validate ok
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: "#/components/schemas/SendVerificationForm"
                  code:
                    type: integer
        '400':
          description: validate failure
          content: 
            application/json:
              schema:
                $ref: "#/components/schemas/CommonResponse"
  "/api/v1/register":   
    post:
      summary: create a new account
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/LoginForm"
        required: true
      responses:
        '201':
          description: create account success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: "#/components/schemas/User"
                  code:
                    type: integer              
        '409':
          description: Email has been registered
          content: 
            application/json:
              schema:
                $ref: "#/components/schemas/CommonResponse"
  "/api/v1/articles":
    post:
      summary: create a new article
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ArticleCreateForm"
        required: true
      responses:
        '201':
          description: create account success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: "#/components/schemas/Article"
                  code:
                    type: integer
  "/api/v1/articles/{articleid}":
    get:
      summary: get article detail
      parameters:
      - name: articleid
        in: path
        description: id of article
        required: true
        schema:
          type: string
      responses:
        '200':
          description: get article sucess
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: "#/components/schemas/Article"
                  code:
                    type: integer
        '400':
          description: article no exist
          content: 
            application/json:
              schema:
                $ref: "#/components/schemas/CommonResponse"
```

新增组件

```yaml
 ArticleCreateForm:
      type: object
      properties:
        title:
          type: string
          maximum: 64
        content:
          type: string
    Article:
      type: object
      properties: 
        id:
          type: string
          maximum: 36
        title:
          type: string
          maximum: 64
        content:
          type: string
        create_at:
          type: string
          format: date-time
        updateed_at:
          type: string
          format: date-time
        status:
          type: integer
        author_id:
          type: integer
          format: int64
          minimum: 1
```

