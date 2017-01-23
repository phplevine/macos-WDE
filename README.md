## macOS 10.12 Sierra Web开发环境配置

主要包括安装Apache、php多个版本、mysql等服务

## 安装Homebrew

在安装之前，请先确认你的系统中是否已经安装了xcode，如果未安装则Homebrew自动帮你安装xode工具包

终端下执行命令安装Homebrew
```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

如果安装成功在命令下输入`$ brew --version`则会显示类似如下信息
```
Homebrew 1.1.7
Homebrew/homebrew-core (git revision 07ce; last commit 2017-01-23)
```

为了确保Homebrew正常工作还需要输入`$ brew doctor`来进行自检

输入命令来tap第三方仓库
```
$ brew tap homebrew/dupes
$ brew tap homebrew/versions
$ brew tap homebrew/php
$ brew tap homebrew/apache
```

## 安装Apache

最新的macOS 10.12 Sierra预装了Apache 2.4，然而，使用这个版本与Homebrew不再是一个简单的任务，因为苹果已经删除了这个版本中的一些必需的脚本。 但是，解决方案是通过Homebrew安装Apache 2.4，然后将其配置为在标准端口（80/443）上运行。

1、停止Apache相关服务
  ```
  $ sudo apachectl stop
  $ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
  ```

2、使用brew安装最Apache24
  ```
  $ brew install httpd24 --with-privileged-ports --with-http2
  ```
  此步骤需要一点时间，因为它从源代码构建Apache，完成后您应该看到一条消息：
  ```
  🍺  /usr/local/Cellar/httpd24/2.4.25: 212 files, 4.5M, built in 2 minutes 45 seconds
  ```

  如果需要开机启动服务，可以输入命令
  ```
  $ sudo cp -v /usr/local/Cellar/httpd24/2.4.25/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons
  $ sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
  $ sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
  $ sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
  ```

  至此完成了Apache的安装，执行命令`$ apachectl start`在浏览器里输入：`http://localhost`会看到如下信息
  ![it-workis](img/it-works.png)

3、如果您收到一条消息，指出浏览器无法连接到服务器，请首先检查以确保服务器已启动。
  ```
  $ ps -aef | grep httpd
  ```
  如果Apache已启动并正在运行，您应该会看到几个httpd进程。
  尝试使用以下命令重新启动Apache：`$ sudo apachectl -k restart`
  在重新启动期间，您可以在新的“终端”选项卡/窗口中查看Apache错误日志，以查看任何内容是否无效或导致问题：`$ tail -f /usr/local/var/log/apache2/error_log`
  如果这不工作，请检查以确保您在/usr/local/etc/apache2/2.4/httpd.conf配置文件中具有Listen：80。

  Apache是通过apachectl命令控制的，所以一些有用的命令要使用：
  ```
  $ sudo apachectl start
  $ sudo apachectl stop
  $ sudo apachectl -k restart
  ```

4、配置Apache
  输入`$ open -e /usr/local/etc/apache2/2.4/httpd.conf`会看到如下信息
  ![textedit](img/textedit.png)

  搜索`DocumentRoot`将`DocumentRoot "/usr/local/var/www/htdocs"`替换成你的路径
  搜索`<Directory /usr/local/var/www/htdocs>`替换成你的路径
  搜索`AllowOverride`改为`AllowOverride All`
  搜索`mod_rewrite`去掉前面注释改为`LoadModule rewrite_module libexec/mod_rewrite.so`
  搜索`User`和`Group`可以将`User`改为你的用户名或者`www`，可以将用户组改为`staff`或者`www`

## 安装PHP
