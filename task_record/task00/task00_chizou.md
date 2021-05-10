# mac下环境搭建

## mysql

官网下载dmg，一路next，设置root密码，设置里已自动勾选开机自启。

连接mysql：

`/usr/local/mysql/bin/mysql -u root -p`

命令行输入：

-- 创建bluewhale用户

`CREATE USER bluewhale@'%' IDENTIFIED BY 'bluewhale';`

`CREATE USER bluewhale@'localhost' IDENTIFIED BY 'bluewhale';`



-- 创建bluewhale数据库，使用UTF8MB4字符集

`CREATE DATABASE bluewhale CHARACTER SET UTF8MB4 COLLATE UTF8MB4_GENERAL_CI;`



-- 授权用户对数据库的服务

`GRANT ALL PRIVILEGES ON bluewhale.* TO 'bluewhale'@'%';`

`GRANT ALL PRIVILEGES ON bluewhale.* TO 'bluewhale'@'localhost';`

`FLUSH PRIVILEGES;`



断开连接后：

`exit；`

`/usr/local/mysql/bin/mysql -u bluewhale -p`

输入密码： bluewhale



查看数据库：`show databases；`



## 后端

本机已有python3.7环境，通过多环境管理。

切换到项目的backend文件夹下。

安装pipenv，python3.8

`brew install pipenv`

激活环境：`pipenv shell`

此后可以看到命令行前面出现（backend）

安装python3.8: `pipenv install --python 3.8`

同步依赖包（会下载）：`pipenv sync` #同步下载项目模块，不需要额外下载django

初始化数据表：`python manage.py migrate`



查看数据表bluewhale：

`use bluewhale;`

`show tables;`



初始化超级管理员用户：`python manage.py createsuperuser`

输入：zymb_1704@126.com / 17351735



启动后端服务：`python manage.py runserver`



## 前端

进入项目文件夹client

本地已安装nodejs14.16，不再处理。

同步依赖包：`npm install `

启动前端：`npm run serve`



成功命令行显示：

>  DONE Compiled successfully in 22787ms               下午11:00:50
>
> 
>
> 
>
>  App running at:
>
>  \- Local:  http://localhost:**8080**/ 
>
> -省略马赛克
>
>  Note that the development build is not optimized.
>
>  To create a production build, run npm run build.

## 调试

浏览器输入：http://localhost:**8080**/

