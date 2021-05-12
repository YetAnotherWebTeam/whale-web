# 理解任务

使用工具（swagger），使用OpenAPI编写RESTful接口：

1. 教程用docker映射镜像/tmp目录，队小伙伴建议有两种方法使用swagger工具：
   -  [https://editor.swagger.io/](https://editor.swagger.io/。) 在线编辑
   - VS code 下载组件OpenAPI，使用

2. 查看代码url.py文件，查看接口函数
3. openapi文档中编辑接口内容
4. 通过工具将接口规范文档转成mock server提供给前端：`npm run mock`
5. swagger页面，点击一个接口，**Try it out** - **Execute**，编辑器将返回mock server的请求返回信息
6. 完成接口规范文档的编写后，重启mock server。并可以通过运行`API_PORT=4010 npm run serve`，使前端项目以mock server为接口进行启动



# OpenAPI文档

- tomo提醒，schema涉及资源的问题。

1. "/api/v1/verify/token"

```python
@api_view(['GET'])
def verify_verification_token(request, token):
    email = validate_token(token)
    if not email:
        return Response({"data": None, "code": 400}, status=status.HTTP_400_BAD_REQUEST)
    return Response({"data": email, "code": 0})
```

GET:

400: email不存在,返回 data，code

200: 正常，返回email，code（同SendVerificationForm格式）



2. "api/v1/register"

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

POST：传入email,passwd

201：正常，返回user，code

400：“message“原因，返回data，code，message

401：“has been registered”，返回data，code，message



3. "/api/v1/articles"

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

        
class Article(models.Model):
    class Status(models.IntegerChoices):
        normal = 0
        deleted = 1
        blocked = 2

    id = models.CharField(_('id'), primary_key=True, max_length=36)
    author = models.ForeignKey(User, on_delete=models.CASCADE, null=False)
    title = models.CharField(_('title'), max_length=64, blank=False)
    content = models.TextField(_('content'), blank=False)
    created_at = models.DateTimeField(_('created at'), auto_now_add=True)
    updated_at = models.DateTimeField(_('modified at'), auto_now=True)
    status = models.IntegerField(_('status'), choices=Status.choices, default=Status.normal)
```

POST：传入id, title,contentm,status

201：正常，返回Article类的各属性



4. "/api/v1/articles/pk"

```python
class ArticleDetailView(BasicRetrieveUpdateDestroyAPIView):
    permission_classes = [IsAuthenticated|ReadOnly]
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

POST：传入id, title,contentm,status

201：正常，返回Article类的各属性

```
openapi: 3.0.3
info:
  title: Bluewhale
  description: 'This is API specifications for bluewhale site'
  version: 1.0.0
servers:
  - url: http://127.0.0.1:4010
paths:
  "/api/v1/me":
    get:
      summary: get current user's profile
      responses:
        '200':
          description: current user
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: "#/components/schemas/User"
                  code:
                    type: integer
  "/api/v1/login":
    options:
      summary: get csrf token
      responses:
        '200':
          description: options
    post:
      summary: login
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/LoginForm"
      responses:
        '200':
          description: success login
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: "#/components/schemas/User"
                  code:
                    type: integer
  "/api/v1/logout":
    post:
      summary: logout
      responses:
        '200':
          description: success logout
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
  
  "/api/v1/verify/token":
    get:
      summary: verify verification token
      responses:
        '200':
          description: success verify token
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
          description: fail verify token
          content:
            application/json:
              schema:
               $ref: "#/components/schemas/CommonResponse"

  "/api/v1/register":
    post:
      summary: register
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/LoginForm"
        required: true 
      responses:
        '201':
          description: right message
          content:
            application/json:
              schema:
                type: object
                properties:
                  data: 
                    $ref: "#/components/schemas/User"
                  code:
                    type: integer
        '400':
          description: wrong token
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: string
                  code:
                    type: integer
                  message:
                    type: string
        '409':
          description: has been registered
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: string
                  code:
                    type: integer
                  message:
                    type: string
  "/api/v1/articles":
    post:
      summary: articles
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Article-post"
        required: true 
      responses:
        '201':
          description: success create articles
          content:
            application/json:
              schema:
                type: object
                properties:
                  data: 
                    $ref: "#/components/schemas/Article"
                  code:
                    type: integer
  "/api/v1/articles/pk":
    get:
      summary: article's detail
      responses:
        '200':
          description: get article's detail
          content:
            application/json:
              schema:
                type: object
                properties:
                  data: 
                    $ref: "#/components/schemas/Article"
                  code:
                    type: integer


components:
  schemas:
    CommonResponse: # common response which has data and code properties
      type: object
      properties:
        data:
          type: object
        code:
          type: integer
    LoginForm:
      type: object
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          format: password
    SendVerificationForm:
      type: object
      properties:
        email:
          type: string
          format: email
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
    Group:
      type: object
      properties:
        id:
          type: integer
          format: int64
          minimum: 1
        name:
          type: string
    Article-post:
      type: object
      properties:
        id:
          type: string
        title:
          type: string
        content:
          type: string
        status:
          type: integer
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

# 实验

- execute后，返回结果与代码相符（反逻辑，先设计接口，再写代码）
- 命令行启动的mock server 返回信息

![1620832821285](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1620832821285.png)

- 前端项目启动