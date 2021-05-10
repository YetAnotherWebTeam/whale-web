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
>  -省略马赛克
>
>  Note that the development build is not optimized.
>
>  To create a production build, run npm run build.

发现与远程master的文件不同

使用nvm管理

先卸载node：因为是官网下载

sudo rm -rf /usr/local/{bin/{node,npm},lib/node_modules/npm,lib/node,share/man/*/node.*}

参考：https://blog.csdn.net/shiquanqq/article/details/78032943

安装nvm：

使用brew

```
brew install nvm
```

根据提示，创建 `.nvm` 目录

```
% mkdir ~/.nvm
```

编辑 `~/.zshrc` 配置文件

```
% vi ~/.zshrc
```

在 `~/.zshrc` 配置文件后添加如下内容

```
export NVM_DIR="$HOME/.nvm"
[ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh"
[ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"
```

`:wq` 保存并退出。

使用 `source` 命令使配置生效

```
% source ~/.zshrc
```

查看一下配置是否生效

```
% echo $NVM_DIR
/Users/your-username/.nvm
```

参考：https://qizhanming.com/blog/2020/07/29/how-to-install-node-using-nvm-on-macos-with-brew

安装

`nvm install 14.15`

使用

```
nvm install --lts # 下载最新的稳定版
nvm use <版本号> # 临时切换版本
nvm alias default <版本号> #永久切换版本（版本别名，default就是默认使用的版本）
nvm uninstall <版本号> # 删除指定版本
nvm ls # 查看本地所有版本
nvm ls-remote --lts # 查看线上所有稳定版 
```

## 调试

浏览器输入：http://localhost:**8080**/

